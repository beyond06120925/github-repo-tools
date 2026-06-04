# GitHub Repo Tools

> Evaluate and compare GitHub repositories — for AI coding assistants

[English](#english) | [中文](#中文)

---

## English

### What is this?

Two complementary tools for GitHub repository analysis:

#### 1. 🔍 Repo Evaluator (`github-repo-evaluator/`)

**5-step structured evaluation** for any repository:

```
┌─────────────────────────────────────────┐
│ Step 1: What is it?                     │
│   → Description, language, use case     │
├─────────────────────────────────────────┤
│ Step 2: Is it useful?                   │
│   → Stars, growth, duplicate detection  │
├─────────────────────────────────────────┤
│ Step 3: What do I need?                 │
│   → API keys, dependencies, costs       │
├─────────────────────────────────────────┤
│ Step 4: How does it work?               │
│   → Before/after workflow comparison    │
├─────────────────────────────────────────┤
│ Step 5: Install & test                  │
│   → Full setup and verification         │
└─────────────────────────────────────────┘
```

**Key feature**: Duplicate detection — checks if you already have similar capabilities installed.

#### 2. ⚖️ Repo Comparator (`github-repo-compare/`)

**Parallel evaluation** of two repositories with 7 dimensions:

| Dimension | What it compares |
|-----------|------------------|
| Functionality | Feature overlap analysis |
| Popularity | Stars, growth rate |
| Maintenance | Update frequency, issue response |
| Documentation | Quality, completeness |
| Cost | API keys, pricing |
| Dependencies | Installation complexity |
| Conflicts | Overlap with existing tools |

**Key feature**: Parallel sub-agent evaluation — evaluates both repos simultaneously.

### Installation

```bash
git clone https://github.com/beyond06120925/github-repo-tools.git

# Copy both skills
cp -r github-repo-tools/github-repo-evaluator ~/.claude/skills/
cp -r github-repo-tools/github-repo-compare ~/.claude/skills/
```

### Usage

**Evaluator**:
- "https://github.com/some/repo — is this worth installing?"
- "Analyze this library for me"
- "帮我看看这个库有没有用"

**Comparator**:
- "Compare React and Vue for my project"
- "Which is better, A or B?"
- "对比这两个仓库哪个更好"

---

## 中文

### 这是什么？

两个互补的 GitHub 仓库分析工具：

1. **仓库评估器** — 5 步结构化评估
2. **仓库对比器** — 并行 7 维度对比

### 使用方法

```bash
git clone https://github.com/beyond06120925/github-repo-tools.git
cp -r github-repo-tools/github-repo-evaluator ~/.claude/skills/
cp -r github-repo-tools/github-repo-compare ~/.claude/skills/
```

---

## License

MIT
