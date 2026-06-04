---
name: github-repo-evaluator
description: "Use when the user provides a GitHub URL and asks to evaluate it — such as '帮我看看', '研究一下', '分析这个库', '这个有没有用'. Triggers on any GitHub repository URL shared with intent to understand, assess, or decide whether to install it."
---

# GitHub Repo Evaluator

## Core Principle

When the user shares a GitHub repository URL with the intent to understand what it does, whether it's useful, and whether to install it — guide them through a structured 5-step evaluation process instead of immediately exploring the codebase.

## The 5-Step Evaluation Process

```
┌─────────────────────────────────────────────────────┐
│  Step 1: 这是什么                                         │
│     gh api repos/{owner}/{repo}                       │
│     gh api repos/{owner}/{repo}/readme                │
│     → 功能描述、适用场景、语言、★                      │
├─────────────────────────────────────────────────────┤
│  Step 2: 有没有用                                         │
│     ★ 总星 / 近期增长 / 同类对比                        │
│     ⚠️ 是否已有类似能力（skill / MCP / 插件）           │
│     → 给出明确推荐（安装 / 不安装 / 先看看）         │
├─────────────────────────────────────────────────────┤
│  Step 3: 需要准备什么                                     │
│     API Key？（哪种、多少、是否付费、哪里弄）          │
│     环境依赖？（Node/Python/Docker）                   │
│     其他准备？（账号/权限/配置）                       │
│     → 明确告知是否付费、多少钱、哪里申请              │
├─────────────────────────────────────────────────────┤
│  Step 4: 原理是什么                                     │
│     装之前 vs 装之后的工作流程对比                    │
│     → 用文字或 ASCII 图展示差异                        │
├─────────────────────────────────────────────────────┤
│  Step 5: 安装配置测试交付                                │
│     确认安装 → 按官方文档安装 → 配置 → 测试 → 交付  │
│     告知触发方式和具体用法                            │
└─────────────────────────────────────────────────────┘
```

## Step 1 — 这是什么

**使用 GitHub MCP 工具：**

```
1. mcp__github__get_file_contents 获取:
   - owner: {owner}
   - repo: {repo}
   - path: "README.md"

2. mcp__github__get_file_contents 获取:
   - owner: {owner}
   - repo: {repo}
   - path: "package.json"

3. mcp__github__get_file_contents 获取:
   - owner: {owner}
   - repo: {repo}
   - path: "skills" (如果存在，判断是否是 Claude Code 技能库)
```

**分析要点：**
- 仓库描述和 README 摘要
- 主要语言（从 package.json 或文件类型判断）
- 功能定位（工具/技能/框架/应用）
- 适用场景

**分析要点：**
- 仓库描述和 README 摘要
- 主要语言
- 功能定位（工具/技能/框架/应用）
- 适用场景

## Step 2 — 有没有用

**评估维度：**
- ★ 总星数（相对同类型库算高还是低）
- 最近增长情况（如果有）
- 和你工作场景的匹配度
- **⚠️ 是否已有类似能力（重点检测）**

**⚠️ 重复能力检测 — 必须执行：**

```bash
# 检查已安装的 skill
ls ~/.claude/skills/ 2>/dev/null

# 检查 installed_plugins.json
cat ~/.claude/plugins/installed_plugins.json 2>/dev/null

# 检查 settings.json 里的 enabledPlugins 和 mcpServers
cat ~/.claude/settings.json 2>/dev/null
```

**检测结果判断：**

| 检测结果 | 处理方式 |
|----------|----------|
| 功能完全重复 | ⭐⭐ 不推荐，说明已有 XXX 提供相同能力 |
| 功能高度重叠 | ⭐⭐⭐ 谨慎推荐，说明和 XXX 功能重叠 XX% |
| 功能互补 | ⭐⭐⭐⭐⭐ 强烈推荐，说明 XXX + 这个 = 完整方案 |
| 全新能力 | ⭐⭐⭐⭐⭐ 强烈推荐 |

**给出明确推荐：**
- ⭐⭐⭐⭐⭐ 强烈推荐安装
- ⭐⭐⭐⭐ 推荐，但可选（需说明和已有能力的关系）
- ⭐⭐⭐ 可先观望
- ⭐⭐ 不推荐，说明原因

## Step 3 — 需要准备什么

**必查项：**
1. **API Key** — README 里通常会写需要什么 key、哪种、多少钱
2. **环境依赖** — Node / Python / Docker / Go 等
3. **账号/权限** — GitHub OAuth / npm token / API token 等
4. **费用** — 免费 / Freemium / 付费（多少钱/月）

**信息来源：** README 的 Installation / Setup / Requirements / Pricing 章节

**告知格式：**
```
准备项          需求           费用          申请地址
─────────────────────────────────────────────────────
API Key        Anthropic      免费额度      console.anthropic.com
Python         3.11+         免费          python.org
```

## Step 4 — 原理是什么

**装之前 vs 装之后对比（图表）：**

```
                        没有这个工具                    有这个工具
                        ──────────────────────────────  ─────────────────────────────
  用户说：              "帮我看看/研究一下/要不要装"     "帮我看看/研究一下/要不要装"
                        │
                        ▼                              ▼
  Claude 行为：        读 README → 摘要给你              自动走 5 步评估
                        │                              │
                        ▼                              ▼
  你得到的：           一段文字描述                     完整结构化报告
                        （不知道有没有用）               （明确推荐度 + 准备项）
                        （不知道怎么装）                 （知道怎么装/配/测/用）
                        （不知道多少钱）                 （知道多少钱/哪里申请）
                        （不知道怎么触发）                （知道触发词 + 输出结果）
                        ──────────────────────────────  ─────────────────────────────
  结论：               自己判断、自己摸索                全流程搞清楚再决定装不装
```

**核心价值：** 节省你反复研究的时间，减少安装后用不起来的情况。

**示例：graphify 触发/结果说明**

| 维度 | 内容 |
|------|------|
| **触发命令** | `/graphify <目标>`（目标可以是代码库路径/文件/URL/文档） |
| **何时触发** | 想理解一个代码库结构、找架构决策的"why"、需要团队共享上下文时 |
| **执行过程** | tree-sitter AST 本地解析 → Whisper 本地转录视频/音频 → Claude 子代理提取语义 → Leiden 社区检测 → 输出图谱 |
| **得到什么** | `graphify-out/` 目录：graph.html（交互图）+ GRAPH_REPORT.md（1页摘要）+ graph.json（可查询图谱） |
| **再次提问时** | 读 GRAPH_REPORT.md（1页）而非全部原始文件，token 减少 71.5x |

## Step 5 — 安装配置测试交付

### 5a. 确认安装
- 问用户：「确认要装吗？」或「先看看别的？」
- 如果用户说算了，停在 Step 4，**不要擅自安装**

### 5b. 安装
- **优先选择 Claude Code 适用方式**（按官方文档顺序）
- 如果有多种安装方式（npm / marketplace / git clone），**优先推荐 Claude Code 原生方式**
- 告知需要什么前置条件

### 5c. 配置
- 读写 `~/.claude/settings.json` 或对应配置文件
- 如果需要环境变量，告知用户要加什么

### 5d. 测试

**自动验证流程（必须执行，重试机制）：**

```
测试次数 = 0
循环直到测试通过 或 测试次数 > 3：

  测试次数 += 1
  执行 skill 的测试命令
  如果测试通过 → 测试通过，进入交付
  如果测试失败：
    如果 测试次数 <= 3 → 调试问题 → 重新执行测试
    如果 测试次数 > 3 → 报错，报错内容：
      • 执行的安装命令
      • 遇到的具体问题
      • 出错时的症状描述
```

**Skill 安装的验证方式：**

| 类型 | 验证方式 |
|------|----------|
| slash command skill | `echo "测试" \| claude -p` 确认 skill 在列表中出现 |
| plugin | 检查 `~/.claude/plugins/installed_plugins.json` 含该 plugin |
| MCP | 检查 `~/.claude/settings.json` 的 mcpServers |
| CLI binary | 执行 `--version` 或 `--help` 确认可运行 |

**报错格式（超过 3 次重试仍失败）：**
```
❌ 安装失败（已重试 3 次）

执行的命令：<命令>
遇到的问题：<具体症状>
建议：<可尝试的解决方案>
```

### 5e. 交付 — 触发方式 + 使用说明 + 结果

对每个 skill，必须提供以下 4 项信息：

| 维度 | 说明 |
|------|------|
| **触发逻辑** | 手动 `/命令` 还是关键字自动触发 |
| **触发命令** | 具体的 slash command（可复制粘贴） |
| **适用场景** | 什么时候用这个 skill（尽量具体，2-3 个真实场景） |
| **得到什么** | 具体输出什么（文件/命令输出/界面变化） |

**触发逻辑说明：**

| 类型 | 触发方式 | 说明 |
|------|----------|------|
| **手动 slash 命令** | 你输入 `/命令` | 最常见，skill 安装后在对话框输入 `/` 触发 |
| **skill 内部调用** | skill 自己调用 | gstack 的 `/review` 在 `github-repo-evaluator` 里被触发，就是这种 |
| **关键字自动** | 某些 skill 有预定义关键字触发 | claude-hud 的 `/hud` 就是，skill 自带的 |
| **hook 自动** | settings.json 里配置 PreToolUse 等 hook | 特定条件下自动执行，不需要你输入 |

**大多数 skill 的触发方式是"你输入 `/命令`"，不是关键字自动触发。**

**示例格式：**

```
【graphify】

触发逻辑：  手动 slash 命令 — 你在对话框输入 /graphify <目标>

触发命令：  /graphify <目标>
          目标 = 路径（.）、文件（a.py）、URL、文档路径

适用场景：  • 第一次接触一个陌生代码库，想快速理解结构
          • 代码库太大（100+ 文件），想找关键架构
          • code review 时想知道这段代码"为什么这样设计"

得到什么：  graphify-out/
            ├── graph.html       # 交互式知识图谱（浏览器打开）
            ├── GRAPH_REPORT.md   # 1页结构化摘要（给 Claude 读）
            └── graph.json       # 可程序化查询的图谱数据
```

**交付标准：**
- 触发命令必须可复制粘贴运行
- 适用场景必须具体到 2-3 个真实使用场景
- 得到什么必须说清楚输出文件名字和格式
```

## Trigger Keywords

**触发方式：任何 GitHub URL + 以下任意关键词/句式**

| 类别 | 关键词 |
|------|--------|
| 调研评估 | 帮我看看, 研究一下, 帮我调研, 调研, 分析这个库, 这个有没有用, 要不要装, 帮我评估, 这个库是什么, 查看, 搜搜, 搜一下 |
| 了解意图 | 这是什么, 了解, 认识, 看看这个, 查一下, 查查 |
| 决策意图 | 装不装, 值不值, 好用吗, 怎么样, 能不能用 |

**兜底规则：**
- 只要用户发的是 GitHub URL 且含有了解/调研/评估意图，自动触发
- 不在列表里的关键词，但用户明显在问 GitHub 仓库的事，也触发

## Important Rules

- **即使发现已安装，仍必须走完完整 5 步评估** — 不能因为"已安装"就跳过 Step 1-4
- **Step 5 的"确认安装"必须在 Step 4 之后** — 用户看完原理和对比后才能决定要不要装
- **即使已安装，也要告知如何触发/使用** — 用户可能还不知道怎么用
- **必须等用户明确说"装"，才能进入 Step 5**

- **不要**不问用户就擅自安装 — 必须等用户确认
- **不要**用 WebFetch — 用 GitHub MCP 工具查数据
- **不要**只说"有用/没用" — 要给出结构化报告
- **不要**忽略费用问题 — 很多库标注免费但实际要付费 API Key

## Red Flags（Claude 容易找借口不走流程时）

| 想偷懒 | 正确做法 |
|--------|---------|
| "README 已经很清楚了，不需要5步" | 必须完整走5步，用户要的是完整报告 |
| "应该有用，直接装吧" | 必须等用户确认 |
| "看看就行，不用测" | 必须验证安装成功才能交付 |
| "这是免费工具" | 必须说清楚免费额度多少、超出怎么收费 |
| "用 npx skills add 就行" | 必须先检查 README 是否有 Claude Code 专属安装方式（`/plugin marketplace add` + `/plugin install`），有就优先用那个 |

## ⚠️ Claude Code 专属安装方式注意点

很多 skill/plugin 的 README 会列出多种安装方式：

1. **通用方式**：`npx skills add xxx/xxx --skill xxx -g`
   - 只安装 SKILL.md
   - 无 autocomplete 命令
   - 触发需要完整命令 `/skill-name:command`

2. **Claude Code 专属方式**：README 里可能有 `For Claude Code-specific features...` 部分
   ```
   /plugin marketplace add xxx/xxx
   /plugin install xxx@xxx
   ```
   - 安装 Plugin + Skill + Hooks
   - 有 autocomplete 命令（如 `/plan` 可简写）
   - 功能更完整

**⚠️ 必须检查 README 是否有 Claude Code 专属安装方式，有就优先推荐那个。**

**Why:** 通用方式安装后功能可能不完整（缺少 hooks、缺少 autocomplete），用户体验差。

**How to apply:** Step 5b 安装时，先 grep README 里是否有 "Claude Code Plugin" 或 "Claude Code-specific" 关键词。
