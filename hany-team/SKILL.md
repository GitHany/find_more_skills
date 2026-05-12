---
name: hany-team
description: Use when wanting a multi-expert team to comprehensively analyze the current project - dispatches parallel expert agents (architect, security, performance, UX, testing, DevOps, code-quality, product) to identify expansion opportunities, optimizations, and bugs, then consolidates into a structured report document
author: hany
version: "1.0"
level: meta
tags: [team, analysis, expert-review, optimization, bug-detection, expansion]
---

# Hany Team - 多专家团队项目分析

> 并发派遣专家 Agent，从多维度深度分析项目，输出结构化报告

## 核心原则

**每个专家独立分析，不互相干扰；最终由 Lead 聚合去重，输出一份完整报告。**

## 流程

```
扫描项目 → 并发派遣 8 位专家 Agent → 聚合去重 → 生成报告文档
```

## 使用场景

- 用户说 "hany-team"、"团队分析"、"专家分析"、"全面审查"、"项目体检"
- 用户想了解项目有哪些可以扩展、优化、修复的点
- 用户想要从不同专家角度审视当前项目
- 用户说 "帮我找 bug"、"帮我找优化点"、"帮我找扩展方向"

## 不适用场景

- 用户有具体明确的单点任务 → 直接执行
- 用户只需要代码审查 → 用 code-review
- 用户只需要安全审查 → 用 security 相关 skill

---

## Phase 1: 项目扫描

在派遣专家之前，Lead 必须先了解项目全貌。

### 扫描内容

1. **项目结构**：目录树、主要模块
2. **技术栈**：语言、框架、依赖（package.json / Cargo.toml / go.mod / pyproject.toml 等）
3. **代码规模**：文件数、代码行数（大致）
4. **测试状态**：测试框架、测试覆盖率（如有）
5. **构建状态**：能否正常构建
6. **Git 状态**：最近提交、分支情况

### 扫描方式

```
1. LS 项目根目录 → 了解顶层结构
2. Glob 关键配置文件 → 识别技术栈
3. Grep 测试框架关键词 → 了解测试情况
4. 读取 README / CLAUDE.md → 了解项目约定
```

### 扫描输出

将扫描结果整理为 `project_context`，作为每个专家 Agent 的共享输入上下文。`project_context` 必须包含：

```markdown
## 项目上下文

- 项目名称：{name}
- 技术栈：{stack}
- 目录结构：{tree}
- 主要模块：{modules}
- 测试框架：{test_framework}
- 代码规模：{scale}
- 关键配置：{configs}
```

---

## Phase 2: 专家团队

### 固定专家团队（8 位）

Lead 直接派遣以下 8 位专家，无需用户确认：

| # | 专家 | 代号 | 分析维度 | 关注点 |
|---|------|------|----------|--------|
| 1 | 架构师 | **Atlas** | 架构设计 | 模块耦合、分层合理性、扩展性、架构模式、依赖关系、循环依赖 |
| 2 | 安全专家 | **Shield** | 安全漏洞 | 注入风险、认证授权、敏感数据泄露、依赖安全、输入验证、CSRF/XSS |
| 3 | 性能专家 | **Turbo** | 性能优化 | 算法复杂度、内存泄漏、N+1 查询、缓存策略、懒加载、bundle 体积 |
| 4 | UX 专家 | **Pixel** | 用户体验 | 交互流畅度、可访问性、响应式、错误提示、加载状态、操作路径 |
| 5 | 测试专家 | **Probe** | 测试质量 | 覆盖率盲区、边界条件、集成测试、E2E、测试可靠性、mock 策略 |
| 6 | DevOps 专家 | **Forge** | 工程效率 | CI/CD、部署流程、监控告警、日志规范、环境管理、容器化 |
| 7 | 代码质量专家 | **Craft** | 代码质量 | 重复代码、命名规范、函数复杂度、死代码、类型安全、错误处理 |
| 8 | 产品专家 | **Spark** | 产品扩展 | 功能扩展点、用户需求盲区、竞品差距、数据驱动机会、API 扩展 |

### 项目自适应

根据项目类型自动调整专家重点：
- 纯后端项目 → Pixel（UX）重点关注 API 响应格式和错误处理
- 无 CI/CD → Forge 侧重"建议搭建 CI/CD"
- 库/SDK 项目 → Spark 重点关注 API 设计和扩展性

---

## Phase 3: 并发派遣专家 Agent

### 派遣方式

使用 `Task` 工具并发派遣所有专家 Agent。每个 Agent 独立运行，互不干扰。

### Agent Prompt 模板

每个专家 Agent 的 prompt 必须包含：

1. **角色定义**：你是谁，你的分析维度
2. **项目上下文**：Phase 1 的 `project_context`
3. **分析要求**：具体要分析什么
4. **输出格式**：必须按统一格式返回

```
你是一位{专家角色}，代号{代号}。你正在从{分析维度}的角度分析一个项目。

## 项目上下文
{project_context}

## 你的任务

从{分析维度}角度深度分析此项目，找出：

1. **Bug / 隐患**：当前代码中存在的 bug 或潜在问题
2. **优化点**：可以改进的性能、结构、模式等
3. **扩展方向**：可以新增的功能、能力、集成等

## 分析要求

- 必须基于实际代码，不要泛泛而谈
- 每个发现必须指明具体文件和代码位置
- 按严重程度排序：Critical > High > Medium > Low
- 尽可能多地发现问题和机会，宁可多不可少
- 不要只看表面，深入分析代码逻辑

## 输出格式

严格按以下 JSON 格式返回：

```json
{
  "expert": "{代号}",
  "dimension": "{分析维度}",
  "findings": [
    {
      "id": "{代号}-001",
      "category": "bug|optimization|expansion",
      "severity": "critical|high|medium|low",
      "title": "简短标题",
      "description": "详细描述",
      "location": "文件路径:行号范围",
      "evidence": "相关代码片段或证据",
      "suggestion": "修复/改进建议",
      "effort": "small|medium|large"
    }
  ],
  "summary": {
    "total": N,
    "bugs": N,
    "optimizations": N,
    "expansions": N,
    "critical_count": N,
    "high_count": N
  }
}
```
```

### 各专家的专属分析指令

#### Atlas（架构师）

额外分析指令：
- 绘制模块依赖图，识别循环依赖
- 评估分层架构是否合理（表现层/业务层/数据层）
- 识别 God Module / God Function
- 评估接口抽象是否恰当
- 分析扩展性：新增模块需要改动多少现有代码
- 检查是否违反 SOLID 原则

#### Shield（安全专家）

额外分析指令：
- 检查所有用户输入点，是否有注入防护
- 检查认证/授权流程
- 检查敏感数据是否暴露在日志/响应中
- 检查依赖是否有已知漏洞
- 检查 CSRF / XSS 防护
- 检查密钥/Token 管理方式
- 检查文件上传/下载安全性

#### Turbo（性能专家）

额外分析指令：
- 识别 O(n²) 及以上复杂度的算法
- 检查是否有 N+1 查询问题
- 检查内存泄漏风险（未清理的监听器、闭包、缓存）
- 评估缓存策略是否合理
- 检查是否有不必要的同步操作
- 分析 bundle 体积和加载性能
- 检查数据库查询是否缺少索引

#### Pixel（UX 专家）

额外分析指令：
- 检查交互反馈是否完整（loading/error/success 状态）
- 评估可访问性（ARIA、键盘导航、对比度）
- 检查响应式设计
- 评估错误提示是否用户友好
- 检查操作路径是否简洁
- 评估动画和过渡是否流畅

#### Probe（测试专家）

额外分析指令：
- 识别无测试覆盖的关键模块
- 检查现有测试的断言质量
- 识别缺少的边界条件测试
- 评估 mock 策略是否合理
- 检查测试是否有副作用（依赖外部服务）
- 评估集成测试 / E2E 测试覆盖

#### Forge（DevOps 专家）

额外分析指令：
- 评估 CI/CD 流程完整性
- 检查部署流程是否自动化
- 评估监控和告警覆盖
- 检查日志规范和聚合
- 评估环境管理策略
- 检查容器化 / 基础设施即代码

#### Craft（代码质量专家）

额外分析指令：
- 识别重复代码（DRY 违反）
- 检查命名规范一致性
- 识别过长函数（>50 行）
- 检查死代码
- 评估类型安全
- 检查错误处理是否完备
- 评估代码注释质量

#### Spark（产品专家）

额外分析指令：
- 识别可以新增的功能扩展点
- 分析用户需求盲区
- 评估 API 设计的扩展性
- 识别数据驱动的机会（分析、报表、推荐）
- 评估与第三方集成的可能性
- 分析竞品差距

---

## Phase 4: 聚合去重

所有专家 Agent 完成后，Lead 进行聚合。

### 聚合步骤

1. **收集所有 Agent 的 JSON 输出**
2. **去重**：相同位置、相同问题的不同专家发现，合并为一条，标注多专家共识
3. **交叉验证**：一个专家发现的 bug，另一个专家可能提供了更深入的根因分析，合并
4. **按严重程度排序**：Critical → High → Medium → Low
5. **按类别分组**：Bug → 优化 → 扩展

### 去重规则

| 情况 | 处理 |
|------|------|
| 不同专家发现同一问题 | 合并为一条，标注 `consensus: [Atlas, Shield]` |
| 同一问题不同视角 | 保留最详细的描述，补充其他视角的 insight |
| 完全独立的问题 | 保留原样 |

### 共识权重

被多个专家同时指出的问题，严重程度自动提升一级（Low → Medium, Medium → High），标注 `consensus_boosted: true`。

---

## Phase 5: 生成报告文档

### 报告路径

`.omc/reports/project-analysis-{timestamp}.md`

如果 `.omc/reports/` 目录不存在，自动创建。

### 报告结构

```markdown
# 项目多专家分析报告

## 元数据

| 项目 | 值 |
|------|-----|
| 项目名称 | {name} |
| 分析时间 | {timestamp} |
| 参与专家 | {expert_list} |
| 发现总数 | {total} |
| Bug 数 | {bug_count} |
| 优化数 | {opt_count} |
| 扩展数 | {exp_count} |

## 摘要

### 严重程度分布

| 严重程度 | Bug | 优化 | 扩展 | 合计 |
|----------|-----|------|------|------|
| Critical | {n} | - | - | {n} |
| High | {n} | {n} | - | {n} |
| Medium | {n} | {n} | {n} | {n} |
| Low | {n} | {n} | {n} | {n} |

### 专家贡献排名

| 排名 | 专家 | 发现数 | Critical | High |
|------|------|--------|----------|------|
| 1 | {expert} | {n} | {n} | {n} |

### Top 10 最重要发现

1. **[{severity}]** {title} — {expert}
2. ...

---

## 详细发现

### 一、Bug / 隐患

#### {id}: {title}

- **严重程度**：{severity}
- **发现者**：{expert} {consensus标注}
- **位置**：{location}
- **描述**：{description}
- **证据**：
  ```
  {evidence}
  ```
- **建议修复**：{suggestion}
- **预估工作量**：{effort}

---

### 二、优化建议

#### {id}: {title}

- **严重程度**：{severity}
- **发现者**：{expert}
- **位置**：{location}
- **描述**：{description}
- **建议方案**：{suggestion}
- **预估收益**：{expected_benefit}
- **预估工作量**：{effort}

---

### 三、扩展方向

#### {id}: {title}

- **优先级**：{severity}
- **发现者**：{expert}
- **描述**：{description}
- **扩展方案**：{suggestion}
- **预估工作量**：{effort}

---

## 各专家原始报告

<details>
<summary>Atlas（架构师）— {n} 项发现</summary>

{atlas_raw_findings}

</details>

<details>
<summary>Shield（安全专家）— {n} 项发现</summary>

{shield_raw_findings}

</details>

...（每位专家一个折叠区域）

---

## 建议执行优先级

### 立即修复（Critical Bug）

1. {bug_title} — {location}
2. ...

### 短期优化（High 优先级）

1. {opt_title} — {location}
2. ...

### 中期规划（Medium 优先级）

1. {item_title}
2. ...

### 长期愿景（Low 优先级 / 扩展方向）

1. {expansion_title}
2. ...
```

---

## 工具使用

- `Task(subagent_type="general_purpose_task")` 并发派遣专家 Agent
- `SearchCodebase` / `Grep` / `Glob` / `Read` 用于项目扫描
- `Write` 写入最终报告

---

## 执行约束

1. **并发派遣**：所有专家 Agent 必须并发派遣，不要顺序执行
2. **独立分析**：每个专家独立工作，不依赖其他专家的结果
3. **基于代码**：所有发现必须基于实际代码，不允许泛泛而谈
4. **具体位置**：每个发现必须指明文件路径和代码位置
5. **尽可能多**：宁可多发现不可少发现，宁抓错不放过
6. **去重不丢失**：聚合时去重，但不丢失任何专家的独特视角
7. **报告完整**：最终报告必须包含所有专家的原始发现（折叠区域）

---

## 常见问题

**Q: 报告太大？**
→ 每个专家的详细发现放在折叠区域，摘要部分保持精简

**Q: 某个专家没有发现？**
→ 仍然在报告中列出该专家，标注"未发现显著问题"
