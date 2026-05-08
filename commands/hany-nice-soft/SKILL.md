---
name: hany-nice-soft
description: 扫描当前项目代码，全面分析警告、Bug 和性能问题，生成 Markdown 诊断报告。当用户说"运行 hany-nice-soft"、"分析项目"、"扫描代码"、"检查警告"、"诊断报告"时使用。
argument-hint: "[--output <path>]"
pipeline: [scan, analyze, report]
---

<Purpose>
Hany-Nice-Soft 是一款全面的代码质量诊断工具，通过系统化扫描项目代码来发现警告、Bug 和性能问题。它帮助开发者在代码重构前进行评估、执行定期质量检查，或在提交前发现潜在问题。工具输出结构化的 Markdown 诊断报告，便于跟踪和修复问题。
</Purpose>

<Use_When>
- 用户说"运行 hany-nice-soft"、"分析项目"、"扫描代码"、"检查警告"、"诊断报告"
- 需要对项目进行全面代码质量诊断时
- 代码重构前的评估
- 定期代码质量检查
- 发现潜在问题和优化点
- 提交代码前的自查
</Use_When>

<Do_Not_Use_When>
- 用户需要修复具体问题而非诊断 — 直接使用编辑器/代码修改工具
- 用户只想查看特定文件的问题 — 直接 grep/搜索该文件
- 用户需要实时监控 — 这是静态分析工具，不是运行时监控
- 用户已有具体的问题列表需要修复
</Do_Not_Use_When>

<Why_This_Exists>
代码质量是软件长期可维护性的基础。手动代码审查耗时且容易遗漏，Hany-Nice-Soft 通过自动化扫描确保一致的检查标准，帮助团队在问题变得严重之前发现它们。它将复杂的静态分析结果转化为可操作的诊断报告，让开发者能够优先级排序修复工作。
</Why_This_Exists>

<Execution_Policy>
- 使用 Grep 和 Glob 工具进行代码扫描
- 严格遵守排除规则，不扫描依赖目录
- 为每个问题分配唯一编号（W001, B001, P001）
- 按严重程度分组输出（Critical, High, Medium, Low）
- 生成结构化的 Markdown 报告
- 不修改任何源代码，只生成诊断报告
- 报告保存到指定路径或输出到控制台
</Execution_Policy>

<Steps>

## Phase 1: 解析参数

1. **解析用户输入**：
   - 检查是否包含 `--output <path>` 参数
   - 确定扫描范围（当前目录或指定目录）
   - 确认报告输出位置

2. **确定工作目录**：
   - 使用当前工作目录作为扫描根目录
   - 如果指定了子目录，合并路径

## Phase 2: 文件扫描

### 2.1 扫描警告 (Warning Detection)

使用 Grep 工具按模式扫描以下内容：

| 检测项 | 搜索模式 | 说明 |
|--------|---------|------|
| TODO/FIXME/HACK/XXX | `TODO\|FIXME\|HACK\|XXX` | 未完成的代码标记 |
| console.log | `console\.log\|console\.debug` | 调试代码残留 |
| debugger | `debugger` | 断点残留 |
| 废弃API使用 | `@deprecated\|Deprecated\|@obsolete` | 废弃标记 |
| 魔法数字 | `\b\d{3,}\b` (需上下文判断) | 硬编码数值 |
| 未使用变量 | 需多工具配合 | 声明但未使用 |

### 2.2 Bug 检测 (Bug Detection)

使用 Grep 工具扫描潜在 Bug 模式：

| 检测项 | 搜索模式 | 说明 |
|--------|---------|------|
| 空指针风险 | `\.length\b\|\.size\b` (需上下文) | 访问可能为 null 的属性 |
| 资源泄漏 | `open\(\|\.close\(\)` | 文件/连接未配对 |
| Promise 未处理 | `new Promise` (无 catch/await) | 异步错误未处理 |
| 硬编码密码 | `password\s*=\s*['"` | 密码硬编码 |
| SQL 注入风险 | `'\s*\+\|`\s*\+` (含 SQL 关键字) | 字符串拼接 SQL |
| XSS 风险 | `innerHTML\s*=` | 直接赋值 HTML |

### 2.3 性能检测 (Performance Detection)

| 检测项 | 搜索模式 | 说明 |
|--------|---------|------|
| 嵌套过深 | 多层 `if\|for\|while` | 超过 3 层嵌套 |
| 同步大文件IO | `readFileSync\|readFile.*sync` | 主线程阻塞 |
| 正则重复编译 | `/.*/` 每次调用 | 正则未缓存 |

## Phase 3: 文件长度检查

使用 Grep 工具检查文件长度：

- **Python 文件**：函数超过 50 行标记为过长
- **其他语言**：函数超过 50 行标记为过长
- **任何文件**：超过 500 行标记为过大

## Phase 4: 生成报告

### 4.1 报告结构

生成包含以下章节的 Markdown 报告：

```markdown
# 代码诊断报告

## 概览摘要
- 扫描时间
- 扫描文件数
- 问题总数
- 质量评分

## 🔴 Critical 问题
| ID | 文件 | 行号 | 问题描述 | 建议修复 |
|----|------|------|----------|----------|

## 🟠 High 问题
...

## 🟡 Medium 问题
...

## 🟢 Low/优化建议
...

## 统计摘要
- 警告数量
- Bug 数量
- 性能问题数量
- 按严重程度分布

## 附录
- 扫描配置
- 排除规则
- 检测规则版本
```

### 4.2 输出方式

- **有 --output 参数**：将报告写入指定文件
- **无 --output 参数**：直接输出到控制台

## Phase 5: 完成

输出完成摘要：
```
诊断完成！发现:
- 🔴 Critical: N 个
- 🟠 High: N 个
- 🟡 Medium: N 个
- 🟢 Low: N 个

报告已保存至: <path>
```

</Steps>

<Tool_Usage>
- 使用 `Grep` 工具进行代码模式扫描
- 使用 `Glob` 工具查找项目文件
- 使用 `Write` 工具生成诊断报告
- 使用 `Read` 工具读取文件上下文
- 不修改任何源代码
</Tool_Usage>

<Examples>
<Good>
用户输入: "运行 hany-nice-soft"
系统行为:
1. 扫描当前目录所有源代码
2. 检测警告、Bug、性能问题
3. 生成 Markdown 诊断报告
4. 输出到控制台

输出:
```
🔴 Critical: 2 个
🟠 High: 5 个
🟡 Medium: 12 个
🟢 Low: 8 个

详见上方诊断报告
```
</Good>

<Good>
用户输入: "hany-nice-soft --output reports/quality.md"
系统行为:
1. 扫描当前目录
2. 生成诊断报告
3. 保存到 reports/quality.md
4. 输出保存路径确认
</Good>

<Bad>
用户输入: "修复这些 console.log"
系统行为: 不应使用 hany-nice-soft
原因: 用户需要修复问题而非诊断，应直接搜索并替换
建议: 使用 Grep 找到所有 console.log，然后批量替换
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- **大量问题**: 如果发现超过 100 个问题，优先报告 Critical 和 High
- **扫描失败**: 如果目录不存在或无权限，显示友好错误信息
- **超时处理**: 如果扫描超过 60 秒仍未完成，输出中间结果并提示继续
- **空结果**: 如果没有发现问题，输出"未检测到明显问题"的正面报告
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] 正确解析 --output 参数
- [ ] 使用 Grep 扫描所有相关模式
- [ ] 排除 node_modules、.git、dist 等目录
- [ ] 为每个问题分配唯一编号
- [ ] 按严重程度正确分组
- [ ] 生成结构完整的 Markdown 报告
- [ ] 报告保存到指定路径或输出到控制台
- [ ] 输出完成摘要统计
- [ ] 不修改任何源代码文件
</Final_Checklist>

<Advanced>

## 排除规则详解

### 排除目录
```
node_modules/, .git/, dist/, build/, target/, out/
__pycache__/, .venv/, venv/, env/
coverage/, .nyc_output/, tmp/, temp/
.vscode/, .idea/, *.log
```

### 排除文件
```
*.min.js, *.bundle.js, *.map.js
package-lock.json, yarn.lock, pnpm-lock.yaml
*.pyc, *.class, *.o, *.obj, *.d.ts
```

## 支持文件类型

| 类别 | 文件类型 |
|------|----------|
| 前端 | .js, .jsx, .mjs, .cjs, .ts, .tsx |
| 后端 | .py, .pyw, .java, .go, .rs, .cs, .php |
| 移动 | .swift, .kt, .kts, .scala |
| 其他 | .html, .css, .sql, .sh, .ps1, .yaml, .json, .xml, .md |

## 严重程度定义

| 等级 | Emoji | 说明 | 示例 |
|------|-------|------|------|
| Critical | 🔴 | 必须立即修复 | 安全漏洞、硬编码密码、致命 Bug |
| High | 🟠 | 尽快修复 | 未处理异常、资源泄漏、空指针风险 |
| Medium | 🟡 | 计划内修复 | TODO/FIXME、过深嵌套、过长函数 |
| Low | 🟢 | 建议优化 | console.log、魔法数字、代码重复 |

## 报告输出格式

```markdown
# 代码诊断报告 - {项目名称}

**扫描时间**: {timestamp}
**扫描范围**: {directory}
**扫描文件**: {count} 个

## 问题统计

| 严重程度 | 数量 |
|---------|------|
| 🔴 Critical | {n} |
| 🟠 High | {n} |
| 🟡 Medium | {n} |
| 🟢 Low | {n} |
| **总计** | {total} |

## 🔴 Critical 问题

| ID | 文件 | 行号 | 问题描述 | 影响分析 | 建议修复 |
|----|------|------|----------|----------|----------|
| C001 | src/auth.js | 42 | 硬编码数据库密码 | 安全风险：密码可能被泄露 | 使用环境变量或密钥管理服务 |

...
```

</Advanced>

Task: {{ARGUMENTS}}
