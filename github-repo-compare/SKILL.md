---
name: github-repo-compare
description: "Use when the user wants to compare two GitHub repositories — such as '对比这两个', '比较一下', '哪个更好', '推荐哪个'. Triggers when user provides 2 GitHub URLs/names with comparison intent."
---

# GitHub Repo Compare

## Core Principle

When the user wants to compare two GitHub repositories to decide which one to install, guide them through a structured comparison process using **parallel sub-agent evaluation**.

## The Comparison Flow

```
┌─────────────────────────────────────────────────────────┐
│  Step 1: 解析输入                                         │
│     解析两个仓库 URL 或名称                                │
│     如果没有提供 → 询问用户                                │
├─────────────────────────────────────────────────────────┤
│  Step 2: 并行评估                                         │
│     Spawn 子代理1 → 评估仓库 A（Step 1-4）                │
│     Spawn 子代理2 → 评估仓库 B（Step 1-4）                │
│     等待两个子代理返回结构化报告                           │
├─────────────────────────────────────────────────────────┤
│  Step 3: 对比分析                                         │
│     对比两个仓库的各维度数据                               │
│     分析功能同质化程度                                     │
│     检测与已安装能力的冲突                                 │
├─────────────────────────────────────────────────────────┤
│  Step 4: 输出推荐                                         │
│     给出明确推荐：A / B / 都不装 / 都装                    │
│     说明推荐理由                                           │
└─────────────────────────────────────────────────────────┘
```

## Step 1 — 解析输入

**提取仓库信息：**

从用户输入中提取两个 GitHub 仓库的标识：
- URL 格式：`https://github.com/{owner}/{repo}`
- 简写格式：`{owner}/{repo}`

**如果没有提供仓库：**

询问用户：「请提供两个要对比的 GitHub 仓库 URL 或名称（如 owner/repo）」

**示例：**

```
用户输入: "对比这两个 https://github.com/A/a 和 https://github.com/B/b 哪个更好"
解析结果:
  - 仓库 A: A/a
  - 仓库 B: B/b

用户输入: "react 和 vue 哪个更适合我"
解析结果:
  - 仓库 A: facebook/react（需要搜索确认）
  - 仓库 B: vuejs/vue（需要搜索确认）
```

## Step 2 — 并行评估

**使用 Agent 工具启动两个 Explore 子代理：**

```
Agent({subagent_type: "Explore", description: "评估仓库 A", prompt: "评估 GitHub 仓库 {owner}/{repo}，执行以下步骤并返回结构化报告..."})

Agent({subagent_type: "Explore", description: "评估仓库 B", prompt: "评估 GitHub 仓库 {owner}/{repo}，执行以下步骤并返回结构化报告..."})
```

**子代理任务（复用 github-repo-evaluator 的 Step 1-4）：**

每个子代理执行：

1. **这是什么** — gh API 获取基本信息、README
2. **有没有用** — 星数、匹配度、重复能力检测
3. **需要准备什么** — API Key、依赖、费用
4. **原理是什么** — 工作流程对比

**子代理返回格式：**

```markdown
## 仓库评估报告：{owner}/{repo}

### 基本信息
| 维度 | 内容 |
|------|------|
| 名称 | xxx |
| 描述 | xxx |
| 语言 | xxx |
| 星数 | xxx |
| 更新时间 | xxx |

### 功能定位
- 功能：xxx
- 适用场景：xxx

### 推荐度
⭐⭐⭐⭐ 推荐

### 准备项
| 准备项 | 需求 | 费用 |
|--------|------|------|
| API Key | xxx | xxx |

### 与已安装能力的关系
- 和 XXX 功能重叠 XX%
- 和 YYY 功能互补
```

## Step 3 — 对比分析

**对比维度：**

| 维度 | 仓库 A | 仓库 B | 对比结论 |
|------|--------|--------|----------|
| 功能定位 | — | — | 是否同质化 |
| 星数 | — | — | 热度对比 |
| 更新频率 | — | — | 维护活跃度 |
| 文档质量 | — | — | 易用程度 |
| 费用 | — | — | 成本对比 |
| 环境依赖 | — | — | 安装难度 |
| 冲突检测 | — | — | 是否与已安装冲突 |

**同质化判断：**

| 重叠程度 | 判断 |
|----------|------|
| 功能完全相同 | 高度同质化 → 推荐更成熟/更活跃的那个 |
| 功能高度重叠 (>70%) | 同质化 → 推荐更适合用户场景的 |
| 功能部分重叠 (30-70%) | 部分同质化 → 说明差异，让用户选择 |
| 功能互补 (<30%) | 不同质化 → 可都装，组合使用 |

**与已安装能力的冲突检测：**

```bash
# 检查已安装的 skill
ls ~/.claude/skills/ 2>/dev/null

# 检查 installed_plugins.json
cat ~/.claude/plugins/installed_plugins.json 2>/dev/null

# 检查 settings.json
cat ~/.claude/settings.json 2>/dev/null
```

## Step 4 — 输出推荐

**推荐类型：**

| 推荐 | 适用场景 |
|------|----------|
| **推荐 A** | A 更成熟、更活跃、更适合用户场景 |
| **推荐 B** | B 更成熟、更活跃、更适合用户场景 |
| **都不装** | 两个都与已安装能力冲突，或都不适合 |
| **都装** | 功能互补，组合使用效果更好 |
| **先观望** | 两个都不够成熟，等后续发展 |

**输出格式：**

```
## 📊 GitHub 仓库对比报告

### 基本信息
| 维度 | 仓库 A ({owner}/{repo}) | 仓库 B ({owner}/{repo}) |
|------|------------------------|------------------------|
| 星数 | ⭐ xxx | ⭐ xxx |
| 语言 | xxx | xxx |
| 更新时间 | xxx | xxx |

### 功能对比
| 维度 | 仓库 A | 仓库 B |
|------|--------|--------|
| 功能定位 | xxx | xxx |
| 适用场景 | xxx | xxx |
| 同质化程度 | — | xxx% |

### 准备项对比
| 维度 | 仓库 A | 仓库 B |
|------|--------|--------|
| API Key | xxx | xxx |
| 环境依赖 | xxx | xxx |
| 费用 | xxx | xxx |

### 与已安装能力的关系
| 仓库 | 冲突情况 |
|------|----------|
| A | 和 XXX 功能重叠 xx% |
| B | 和 YYY 功能重叠 xx% |

### 🎯 推荐

**推荐安装：仓库 A / 仓库 B / 都不装 / 都装**

**理由：**
- xxx
- xxx

**如果要装，下一步：**
- 告知用户具体安装命令
```

## Trigger Keywords

**触发方式：2 个 GitHub URL/名称 + 以下任意关键词**

| 类别 | 关键词 |
|------|--------|
| 对比意图 | 对比这两个, 比较一下, 对比下, 比较, 比一比 |
| 选择意图 | 哪个更好, 推荐哪个, 装哪个, 选哪个, 哪个更适合, 哪个好 |

**兜底规则：**
- 用户提供 2 个 GitHub 相关信息且有对比/选择意图，自动触发
- 只提供 1 个仓库 → 不触发（可能应该用 github-repo-evaluator）
- 不在列表里的关键词，但明显在对比两个仓库，也触发

## Important Rules

- **必须并行评估** — 两个子代理同时启动，节省时间
- **必须等两个子代理返回** — 不能在只有一个返回时就输出推荐
- **必须检测同质化** — 这是核心功能，不能跳过
- **必须检测已安装冲突** — 避免推荐了用户已经有的东西

- **不要**只给一个推荐就结束 — 要说明理由
- **不要**忽略费用差异 — 成本是重要决策因素
- **不要**忽略用户场景 — 技术指标不是唯一标准
- **不要**只给 1 个仓库就用这个 skill → 用 github-repo-evaluator

## Edge Cases

| 场景 | 处理方式 |
|------|----------|
| 只提供 1 个仓库 | 建议用 github-repo-evaluator 或询问另一个仓库 |
| 仓库不存在 | 告知用户仓库 URL 无效，请求正确的 |
| 两个仓库完全相同 | 告知用户这是同一个仓库 |
| 两个仓库功能完全不同 | 说明各自用途，建议根据需求选择或都装 |

## Sub-Agent Prompt Template

**发给子代理的 prompt：**

```
评估 GitHub 仓库 {owner}/{repo}，执行以下步骤并返回结构化报告。

## Step 1: 基本信息
使用 GitHub MCP 工具获取：
- mcp__github__get_file_contents 获取 README.md
- mcp__github__get_file_contents 获取 package.json（如果存在）

返回：名称、描述、语言、星数、更新时间、功能定位

## Step 2: 有没有用
评估：星数热度、适用场景、与你工作匹配度

检测已安装能力：
- ls ~/.claude/skills/
- cat ~/.claude/plugins/installed_plugins.json
- cat ~/.claude/settings.json

返回：推荐度（⭐1-5）、与已安装能力的关系

## Step 3: 需要准备什么
检查 README 的 Installation/Requirements/Pricing 章节

返回：API Key、环境依赖、费用

## Step 4: 原理是什么
描述这个库解决什么问题、如何工作

返回：一句话价值描述

## 输出格式
按以下结构返回，方便汇总：
- 基本信息（表格）
- 功能定位
- 推荐度（星级）
- 准备项（表格）
- 与已安装能力的关系
```