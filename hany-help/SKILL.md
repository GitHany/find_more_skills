---
name: hany-help
description: 根据用户需求智能推荐技能并显示执行管道，然后让用户选择执行
author: hany
version: "6.0"
level: meta
tags: [skill-recommendation, workflow]
next-skill: [find-skills, hany-require, brainstorming]
---

# hany-help - 智能技能推荐助手

> **v6.0** | 动态发现 → 精准评分 → 并发调度 → 简洁执行

## 流程

```
解析需求 → 扫描技能 → 评分排序 → 并发分析 → 生成管道 → 用户选择 → 执行技能
```

| 步骤 | 动作 |
|------|------|
| 0 | 解析需求：意图、领域、关键词 + Agent 直接推荐 |
| 1 | 扫描技能：递归发现 SKILL.md |
| 2 | 评分排序：多维度评分 |
| 2.5 | 并发分析：依赖关系 + 并行标记 |
| 3 | 生成管道：推荐执行路径 |
| 4 | 用户选择：AskUserQuestion |
| 5 | 执行技能：Skill() |

---

## Step 0：解析需求

### 需求结构

```yaml
intent_type: create | optimize | debug | plan | explore | analyze | generate
domain: frontend | backend | test | design | plan | debug | optimize | doc
confidence: high (>15) | medium (5-15) | low (<5)
keywords: [关键词列表]
```

### 意图识别

| 意图 | 关键词 | 推荐技能 |
|------|-------|---------|
| create | 创建, 新建, 写 | brainstorming |
| optimize | 优化, 改进 | code-quality-audit |
| debug | 调试, 修复, fix | systematic-debugging |
| plan | 规划, 计划 | writing-plans |
| explore | 找, 搜索 | find-skills |
| analyze | 分析, 研究 | consulting-analysis |

> 模糊需求 → 推荐 `hany-deep`

### Agent 直接推荐

在解析需求后，检查是否有 agent 类型直接匹配：

```yaml
agent_match:
  frontend: frontend-architecture-performance-expert
  backend: python-architecture-expert
  security: security-vulnerability-scanning-protection-expert
  testing: testing-strategy-expert
  data: python-data-analysis-visualization-expert
  api: restful-api-designer | graphql-api-designer
  performance: performance-optimization-specialist | python-performance-optimizer
  debugging: intelligent-debugging-diagnosis
```

**匹配到时显示**：
```
💡 发现专属 Agent: {agent名} - 直接调用比走技能推荐更快
```

> 仍保留技能推荐供用户选择

---

## Step 1：扫描技能（动态发现）

### 扫描路径

递归扫描以下位置，自动发现含 SKILL.md 的目录：

| 路径类型 | 路径 |
|----------|------|
| 工作目录 | `{cwd}/` |
| 工作目录 | `{cwd}/.claude/skills/` |
| 工作目录 | `{cwd}/.agents/skills/` |
| 用户目录 | `{user_home}/.trae-cn/skills/` |
| 全局目录 | `{cwd}/skills/` |

### 扫描逻辑

```
1. 递归遍历所有路径
2. 查找 SKILL.md 文件
3. 解析 frontmatter 元数据
4. 缓存技能列表（避免重复扫描）
```

### 提取字段

```yaml
name: frontmatter.name
path: 文件路径
description: 描述（>100字符 +4分）
keywords: frontmatter.tags 合并提取
use_when: frontmatter.description 提取触发场景
quality: 0-20分
```

### 质量分

| 元数据 | 分值 |
|--------|------|
| pipeline | +5 |
| next-skill | +5 |
| level | +3 |
| description > 100字符 | +4 |

---

## Step 2：评分排序（扩展公式）

### 评分公式

```
总分 = 关键词匹配(35) + 意图匹配(25) + 领域匹配(25) + 质量分(0-20) + Use_When匹配(15) + 时序权重(±5)
```

### 匹配得分

| 匹配程度 | 意图 | 领域 | Use_When |
|---------|------|------|----------|
| 完全匹配 | 25 | 25 | 15 |
| 部分匹配 | 15 | 15 | 10 |
| 无匹配 | 5 | 5 | 0 |

### 时序权重

最近使用过的技能降权，避免重复推荐：

| 情况 | 权重 |
|------|------|
| 本次会话已用过 | -5 |
| 上次会话用过 | -3 |
| 首次使用 | +2 |

### 评分分级

| 分数 | 评级 | 标记 |
|------|------|------|
| 80-100 | 🎯 强烈推荐 | [🎯] |
| 60-79 | ⭐ 推荐 | [⭐] |
| 40-59 | 👍 一般 | [👍] |
| 0-39 | - 不推荐 | - |

---

## Step 2.5：并发分析

### 依赖分析

分析推荐技能间的依赖关系：

```yaml
independent: [skillA, skillB]  # 无依赖，可并行
sequential: [skillA → skillB]  # 有依赖，必须顺序
```

### 并行标记

| 类型 | 标记 | 管道表示 |
|------|------|----------|
| 可并发 | ⚡ | `{skillA \| skillB}` |
| 需顺序 | → | `skillA → skillB` |

### 并发规则

- 两个技能无共享资源/输出依赖 → 可并发
- 一个技能的输出是另一个的输入 → 需顺序
- 涉及同一文件修改 → 需顺序

---

## Step 3：生成管道

### 管道格式

```
{single_skill}
{skill1 → skill2 → skill3}
{parallelable | skills}
```

### 常用管道

| 场景 | 管道 |
|------|------|
| 需求模糊 | `hany-deep → brainstorming → writing-plans` |
| 有想法 | `brainstorming → writing-plans` |
| 已有计划 | `writing-plans → executing-plans` |
| 调试修复 | `systematic-debugging \| test-driven-development` |
| 前端实现 | `brainstorming → writing-plans → premium-frontend-ui` |
| 并发执行 | `{find-skills \| planning-with-files-zh}` |

---

## Step 4：用户选择

### 输出格式

```
════════════════════════════════════════
需求：{需求文本}

💡 发现专属 Agent: {agent名}

推荐技能：
1. [🎯] {技能名} - {描述}
2. [⭐] {技能名} - {描述}

推荐管道：{skill1} | {skill2}（可并发）
════════════════════════════════════════
```

### 选择菜单

```yaml
- label: "{技能名} {⚡标记}"
- label: "查看全部可用技能"
- label: "搜索新技能"
```

---

## Step 5：执行技能

```python
1. 确认选择 → "✅ 已选择：{技能名}"
2. 调用技能 → Skill("{技能名}")
3. 传递需求 → 原始需求作为参数
```

---

## 边界情况

| 情况 | 处理 |
|------|------|
| 技能 < 3 | 显示全部 |
| 技能 > 10 | 显示 Top 5 |
| 最高分 < 40 | 全部 + 建议 find-skills |
| 并发冲突 | 自动拆分为顺序执行 |

---

## 自检清单

- [ ] 需求已明确（≥5字符）
- [ ] Agent 直接推荐已检查
- [ ] 扫描只执行一次
- [ ] 评分计算正确（含时序权重）
- [ ] 并发分析已完成
- [ ] 管道逻辑合理
- [ ] 用户选择已确认

---

## 调试模式

启用：`/hany:help [需求] debug=true`

---

## FAQ

**Q1: 推荐的技能不准确？**
→ 检查：description 是否清晰、use_when 是否包含触发关键词、level/tags 元数据是否添加

**Q2: 如何让自定义技能被优先推荐？**
→ 在 SKILL.md 添加清晰 description + use_when 触发场景描述

**Q3: 扫描不到技能？**
→ 检查：SKILL.md 是否存在、路径是否在扫描范围内、frontmatter 是否有效

---

## 元数据规范

```yaml
---
name: skill-name
description: 简洁描述（50-200字符）
author: 作者
version: "1.0"
level: frontend  # frontend/backend/meta/general
tags: [关键词]
use_when: 触发场景描述
pipeline: [step1, step2]
next-skill: [other-skill]
---
```
