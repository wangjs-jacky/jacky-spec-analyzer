# spec-analyzer Skill 实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 创建一个 Claude Code skill，用于分析 spec-driven 仓库并提取核心知识

**Architecture:** 使用 Claude Code Skill 框架，通过精心设计的 prompt 指导 Claude 分析文档结构、提取问题域、解决方案和技术要点，输出 Markdown 和可交互 HTML

**Tech Stack:** Claude Code Skill (SKILL.md), Markdown, HTML/CSS (用于 GitHub Pages)

---

## Task 1: 初始化仓库结构

**Files:**
- Create: `README.md`
- Create: `.gitignore`
- Create: `skills/spec-analyzer/SKILL.md` (骨架)

**Step 1: 创建 README.md**

```markdown
# jacky-spec-analyzer

分析 spec-driven 仓库，提取核心知识并生成有条理的文档。

## 功能

- 分析项目要解决的问题
- 提取解决方案和工作流程
- 提取关键技术要点
- 输出 Markdown 和可交互 HTML

## 使用方式

```bash
# 分析远程仓库
/spec-analyzer https://github.com/xxx/yyy

# 分析本地仓库
/spec-analyzer /path/to/repo

# 指定选项
/spec-analyzer https://github.com/xxx/yyy --output ./docs --depth deep
```

## 安装

```bash
j-skills link spec-analyzer
```
```

**Step 2: 创建 .gitignore**

```gitignore
node_modules/
analyzed/
.DS_Store
*.log
```

**Step 3: 创建 skill 目录结构**

```bash
mkdir -p skills/spec-analyzer
```

**Step 4: 初始化 Git 仓库**

```bash
cd /Users/jacky/jacky-github/jacky-spec-analyzer
git init
git add .
git commit -m "init: initial project structure"
```

---

## Task 2: 创建 spec-analyzer Skill 骨架

**Files:**
- Create: `skills/spec-analyzer/SKILL.md`

**Step 1: 编写 SKILL.md 骨架**

```markdown
---
name: spec-analyzer
description: 分析 spec-driven 仓库，提取核心知识。当用户想要分析 GitHub 仓库、提取项目文档、了解项目架构时触发。
---

# spec-analyzer

分析 spec-driven 仓库，提取以下三类核心信息：

1. **问题域** - 项目解决什么问题，之前存在什么痛点
2. **解决方案** - 如何解决这些问题的，核心工作流程
3. **技术要点** - 关键实现细节

## 使用方式

```bash
# 分析远程仓库
/spec-analyzer https://github.com/gsd-build/get-shit-done

# 分析本地仓库
/spec-analyzer /path/to/local/repo

# 指定输出目录和分析深度
/spec-analyzer https://github.com/xxx/yyy --output ./docs --depth deep
```

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--output, -o` | 输出目录 | `./analyzed/` |
| `--depth, -d` | 分析深度 (quick/deep) | `quick` |
| `--lang, -l` | 输出语言 (zh-CN/en) | `zh-CN` |

## 工作流程

### 1. 确定输入源

- 如果是 GitHub URL，克隆到临时目录
- 如果是本地路径，直接使用

### 2. 扫描文档

递归查找所有 `.md` 文件，重点关注：

- `README.md`
- `docs/` 目录
- `*.md` 规范文件（如 `PROJECT.md`, `ROADMAP.md`, `REQUIREMENTS.md`）

### 3. 分析文档

根据分析深度执行不同的分析策略：

**快速模式 (quick)**:
- 读取 README 和主要文档
- 提取核心信息

**深度模式 (deep)**:
- 读取所有 markdown 文件
- 详细分析每个章节
- 提取代码示例和技术细节

### 4. 提取核心信息

按照以下框架提取信息：

#### 问题域

1. 这个项目要解决什么问题？
2. 没有这套系统之前，用户是怎么做的？
3. 之前的做法存在什么痛点？

#### 解决方案

1. 这套系统如何解决这些问题？
2. 核心工作流程是什么？
3. 有哪些关键设计决策？

#### 技术要点

1. 关键技术实现
2. 核心概念和术语
3. 代码示例和用法

### 5. 生成输出

输出到指定目录：

```
./analyzed/<repo-name>/
├── overview.md           # 项目概览
├── problems.md           # 问题分析
├── solutions.md          # 解决方案
├── technical-points.md   # 技术要点
├── structure.json        # 结构化数据
└── html/                 # 可交互 HTML
    ├── index.html
    └── assets/
```

## 输出模板

### overview.md

```markdown
# {项目名称} 概览

## 一句话简介

{项目的一句话描述}

## 核心价值

{这个项目解决了什么核心问题}

## 目标用户

{这个项目是为谁设计的}

## 快速开始

{如何快速上手使用}
```

### problems.md

```markdown
# {项目名称} 问题分析

## 背景问题

{这个项目要解决什么问题}

## 现有方案的痛点

| 痛点 | 描述 | 影响 |
|------|------|------|
| ... | ... | ... |

## 问题根源分析

{为什么会有这些问题}
```

### solutions.md

```markdown
# {项目名称} 解决方案

## 核心思路

{这套系统如何解决问题}

## 工作流程

{详细的工作流程描述}

## 关键设计

{关键设计决策和原因}
```

### technical-points.md

```markdown
# {项目名称} 技术要点

## 核心概念

| 概念 | 说明 |
|------|------|
| ... | ... |

## 关键实现

{关键技术实现细节}

## 代码示例

{代码示例}
```

## 示例

分析 GSD 仓库：

```bash
/spec-analyzer https://github.com/gsd-build/get-shit-done --depth deep
```

预期输出：

- `overview.md`: GSD 是一个 spec-driven development 系统
- `problems.md`: Context Rot 问题、Vibecoding 的不可靠性
- `solutions.md`: Wave 执行、200K 新鲜上下文、需求追踪
- `technical-points.md`: XML Prompt 格式、多 Agent 编排、原子提交
```

**Step 2: 提交骨架**

```bash
git add skills/spec-analyzer/SKILL.md
git commit -m "feat: add spec-analyzer skill skeleton"
```

---

## Task 3: 创建优化工作流文档

**Files:**
- Create: `workflows/optimize-analyzer.md`

**Step 1: 编写优化工作流**

```markdown
# spec-analyzer 优化工作流

## 目的

通过迭代优化 spec-analyzer 的输出质量，使其能够准确提取 spec-driven 仓库的核心知识。

## 工作流程

### 1. 运行分析

```bash
/spec-analyzer <target-repo> --depth deep
```

### 2. 审查输出

检查以下方面：

- [ ] 问题域是否准确提取？
- [ ] 痛点分析是否全面？
- [ ] 解决方案描述是否清晰？
- [ ] 技术要点是否遗漏关键内容？

### 3. 标注问题

在输出文件中添加注释：

```markdown
<!-- ISSUE: 这里应该提到 Wave 并行执行的概念 -->
<!-- ISSUE: 遗漏了需求追踪机制的说明 -->
```

### 4. 更新模板

根据问题更新 `templates/analysis-template.md`：

```markdown
# 更新模板示例

## 问题域

### 核心问题
<!-- 添加: 必须提取项目要解决的核心问题 -->

### 现有痛点
<!-- 添加: 列出之前方案的至少 3 个痛点 -->
```

### 5. 更新 Skill

更新 `skills/spec-analyzer/SKILL.md` 中的分析指令。

### 6. 验证改进

重新运行分析，验证输出质量是否提升。

## 迭代记录

| 日期 | 目标仓库 | 改进点 | 效果 |
|------|----------|--------|------|
| ... | ... | ... | ... |
```

**Step 2: 创建 workflows 目录并提交**

```bash
mkdir -p workflows
git add workflows/optimize-analyzer.md
git commit -m "docs: add optimize-analyzer workflow"
```

---

## Task 4: 创建分析模板

**Files:**
- Create: `templates/analysis-template.md`

**Step 1: 编写分析模板**

```markdown
# 分析模板

此模板定义 spec-analyzer 的输出格式和结构。

## 问题域模板

```markdown
# {项目名称} - 问题分析

## 1. 核心问题

> 一句话描述这个项目要解决的核心问题

**问题描述**: {详细描述}

**影响范围**: {谁受影响，影响多大}

## 2. 背景与现状

### 2.1 之前的做法

{描述在没有这套系统之前，用户是如何处理类似问题的}

### 2.2 现有方案的痛点

| 痛点 | 描述 | 根本原因 | 影响 |
|------|------|----------|------|
| {痛点1} | {描述} | {为什么存在} | {对用户的影响} |
| {痛点2} | {描述} | {为什么存在} | {对用户的影响} |

## 3. 问题根源分析

{分析这些问题的根本原因}
```

## 解决方案模板

```markdown
# {项目名称} - 解决方案

## 1. 核心思路

> 一句话描述这套系统的核心思路

{详细说明}

## 2. 工作流程

### 流程图

{使用 Mermaid 绘制流程图}

### 步骤说明

1. **步骤1**: {说明}
2. **步骤2**: {说明}
...

## 3. 关键设计决策

| 决策 | 原因 | 权衡 |
|------|------|------|
| {决策1} | {为什么这样选择} | {放弃了什么} |
| {决策2} | {为什么这样选择} | {放弃了什么} |

## 4. 与现有方案对比

| 维度 | 现有方案 | 本系统 |
|------|----------|--------|
| {维度1} | {描述} | {描述} |
```

## 技术要点模板

```markdown
# {项目名称} - 技术要点

## 1. 核心概念

| 概念 | 定义 | 用途 |
|------|------|------|
| {概念1} | {定义} | {用途} |
| {概念2} | {定义} | {用途} |

## 2. 关键实现

### 2.1 {实现1}

**目的**: {这个实现解决什么问题}

**原理**: {工作原理}

**代码示例**:
```{language}
{代码}
```

### 2.2 {实现2}

...

## 3. 数据流

{描述数据如何在系统中流动}

## 4. 扩展点

{系统提供了哪些扩展能力}
```
```

**Step 2: 创建目录并提交**

```bash
mkdir -p templates
git add templates/analysis-template.md
git commit -m "docs: add analysis template"
```

---

## Task 5: 创建 HTML 输出模板

**Files:**
- Create: `templates/html/index.html`
- Create: `templates/html/styles.css`

**Step 1: 创建 HTML 模板**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{projectName} - 分析报告</title>
    <link rel="stylesheet" href="assets/styles.css">
</head>
<body>
    <nav class="sidebar">
        <h2>{projectName}</h2>
        <ul>
            <li><a href="#overview">概览</a></li>
            <li><a href="#problems">问题分析</a></li>
            <li><a href="#solutions">解决方案</a></li>
            <li><a href="#technical">技术要点</a></li>
        </ul>
    </nav>
    <main class="content">
        <section id="overview">
            {overviewContent}
        </section>
        <section id="problems">
            {problemsContent}
        </section>
        <section id="solutions">
            {solutionsContent}
        </section>
        <section id="technical">
            {technicalContent}
        </section>
    </main>
</body>
</html>
```

**Step 2: 创建 CSS 样式**

```css
:root {
    --primary: #2563eb;
    --bg: #f8fafc;
    --sidebar-bg: #1e293b;
    --text: #1e293b;
    --text-light: #94a3b8;
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: var(--bg);
    color: var(--text);
    display: flex;
    min-height: 100vh;
}

.sidebar {
    width: 280px;
    background: var(--sidebar-bg);
    padding: 2rem;
    position: fixed;
    height: 100vh;
    overflow-y: auto;
}

.sidebar h2 {
    color: white;
    margin-bottom: 2rem;
    font-size: 1.25rem;
}

.sidebar ul {
    list-style: none;
}

.sidebar li {
    margin-bottom: 0.5rem;
}

.sidebar a {
    color: var(--text-light);
    text-decoration: none;
    padding: 0.5rem 1rem;
    display: block;
    border-radius: 0.5rem;
    transition: all 0.2s;
}

.sidebar a:hover {
    background: rgba(255,255,255,0.1);
    color: white;
}

.content {
    margin-left: 280px;
    padding: 2rem 3rem;
    flex: 1;
    max-width: 900px;
}

section {
    margin-bottom: 4rem;
}

h1, h2, h3 {
    margin-bottom: 1rem;
}

h1 { font-size: 2rem; }
h2 { font-size: 1.5rem; color: var(--primary); }
h3 { font-size: 1.25rem; }

p {
    line-height: 1.8;
    margin-bottom: 1rem;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin: 1rem 0;
}

th, td {
    padding: 0.75rem 1rem;
    border: 1px solid #e2e8f0;
    text-align: left;
}

th {
    background: #f1f5f9;
    font-weight: 600;
}

code {
    background: #f1f5f9;
    padding: 0.2rem 0.5rem;
    border-radius: 0.25rem;
    font-size: 0.9em;
}

pre {
    background: #1e293b;
    color: #e2e8f0;
    padding: 1rem;
    border-radius: 0.5rem;
    overflow-x: auto;
    margin: 1rem 0;
}

pre code {
    background: none;
    padding: 0;
}
```

**Step 3: 创建目录并提交**

```bash
mkdir -p templates/html/assets
git add templates/html/
git commit -m "feat: add HTML output templates"
```

---

## Task 6: 创建 GSD 分析示例目录

**Files:**
- Create: `examples/gsd-analysis/.gitkeep`

**Step 1: 创建示例目录**

```bash
mkdir -p examples/gsd-analysis
touch examples/gsd-analysis/.gitkeep
```

**Step 2: 提交**

```bash
git add examples/
git commit -m "chore: add examples directory for GSD analysis"
```

---

## Task 7: 链接 Skill 到全局

**Step 1: 使用 j-skills 链接**

```bash
cd /Users/jacky/jacky-github/jacky-spec-analyzer
j-skills link spec-analyzer
```

**Step 2: 验证链接**

```bash
j-skills link --list
```

**Step 3: 提交最终状态**

```bash
git add .
git commit -m "chore: finalize project setup"
```

---

## Task 8: 测试 Skill

**Step 1: 启动新的 Claude Code 会话测试**

```bash
# 在新会话中运行
/spec-analyzer https://github.com/gsd-build/get-shit-done --depth deep --output ./examples/gsd-analysis
```

**Step 2: 检查输出**

验证以下文件是否正确生成：
- `overview.md`
- `problems.md`
- `solutions.md`
- `technical-points.md`

---

## 执行总结

| Task | 描述 | 文件 |
|------|------|------|
| 1 | 初始化仓库结构 | README.md, .gitignore |
| 2 | 创建 Skill 骨架 | skills/spec-analyzer/SKILL.md |
| 3 | 创建优化工作流 | workflows/optimize-analyzer.md |
| 4 | 创建分析模板 | templates/analysis-template.md |
| 5 | 创建 HTML 模板 | templates/html/ |
| 6 | 创建示例目录 | examples/gsd-analysis/ |
| 7 | 链接 Skill | - |
| 8 | 测试 Skill | - |
