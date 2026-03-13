---
name: spec-analyzer
description: 分析 spec-driven 仓库，提取核心知识。当用户想要分析 GitHub 仓库、提取项目文档、了解项目架构、翻译项目文档时触发。
---

# spec-analyzer

分析 spec-driven 仓库，提取以下三类核心信息：

1. **问题域** - 项目解决什么问题，之前存在什么痛点
2. **解决方案** - 如何解决这些问题的，核心工作流程
3. **技术要点** - 关键实现细节

## 使用方式

```bash
# 分析远程仓库（自动 clone 并翻译为中文）
/spec-analyzer https://github.com/gsd-build/get-shit-done

# 分析本地仓库
/spec-analyzer /path/to/local/repo

# 指定输出目录和分析深度
/spec-analyzer https://github.com/xxx/yyy --output ./docs --depth deep

# 保留英文原文，不翻译
/spec-analyzer https://github.com/xxx/yyy --lang en
```

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--output, -o` | 输出目录 | `./analyzed/` |
| `--depth, -d` | 分析深度 (quick/deep) | `quick` |
| `--lang, -l` | 输出语言 (zh-CN/en) | `zh-CN` |
| `--keep-repo, -k` | Clone 后保留仓库（不删除） | `false` |

## 工作流程

### 1. 确定输入源并 Clone 仓库

#### 如果是 GitHub URL

**必须先 Clone 仓库到本地临时目录**：

1. 解析 GitHub URL，提取 `owner/repo` 信息
2. 检查是否已配置 Git 代理（如需要）
3. Clone 仓库到临时目录：
   ```bash
   # macOS/Linux 临时目录
   cd /tmp && git clone https://github.com/owner/repo.git
   ```
4. 记录 clone 路径，后续分析使用此路径
5. 分析完成后，根据 `--keep-repo` 参数决定是否删除临时仓库

**Clone 命令示例**：
```bash
# 设置代理（如需要）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# Clone 仓库
cd /tmp
git clone https://github.com/gsd-build/get-shit-done.git

# 完成后取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### 如果是本地路径

- 直接使用提供的路径进行分析
- 跳过 clone 步骤

### 2. 扫描文档

递归查找所有 `.md` 文件，重点关注：

- `README.md`
- `docs/` 目录
- `*.md` 规范文件（如 `PROJECT.md`, `ROADMAP.md`, `REQUIREMENTS.md`, `STATE.md`）

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
4. 问题的根本原因是什么？

#### 解决方案

1. 这套系统如何解决这些问题？
2. 核心工作流程是什么？
3. 有哪些关键设计决策？
4. 与现有方案相比有什么优势？

#### 技术要点

1. 关键技术实现
2. 核心概念和术语
3. 代码示例和用法
4. 数据流和架构

### 5. 文档翻译

**当 `--lang zh-CN` 时（默认），需要将英文文档翻译为中文**：

#### 翻译原则

1. **保持格式不变**：Markdown 结构、代码块、链接、表格等格式保持原样
2. **专业术语保留**：技术术语保留英文或使用中英对照，如：
   - Context Rot → Context Rot（上下文腐化）
   - Wave → Wave（波次）
   - Agent → Agent（代理）
   - PR → PR（Pull Request）
3. **代码不翻译**：代码块、命令、文件名、路径不翻译
4. **链接不修改**：URL 链接保持原样
5. **符合中文习惯**：使用流畅的中文表达，避免机翻感

#### 翻译范围

- README.md
- docs/ 目录下所有 .md 文件
- PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md 等规范文件
- 代码注释（可选，根据深度决定）

#### 翻译示例

**原文**：
```markdown
## Getting Started

First, install the dependencies:

\`\`\`bash
npm install
\`\`\`
```

**翻译后**：
```markdown
## 快速开始

首先，安装依赖：

\`\`\`bash
npm install
\`\`\`
```

### 6. 生成输出

输出到指定目录：

```
./analyzed/<repo-name>/
├── overview.md           # 项目概览（中文）
├── problems.md           # 问题分析（中文）
├── solutions.md          # 解决方案（中文）
├── technical-points.md   # 技术要点（中文）
├── structure.json        # 结构化数据
├── translated/           # 翻译后的原文档（可选）
│   ├── README.md
│   └── docs/
│       └── *.md
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

### solutions.md

```markdown
# {项目名称} 解决方案

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

### technical-points.md

```markdown
# {项目名称} 技术要点

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

## 示例

### 示例 1: 分析并翻译 GitHub 仓库

```bash
/spec-analyzer https://github.com/gsd-build/get-shit-done --depth deep
```

**执行步骤**：
1. Clone 仓库到 `/tmp/get-shit-done/`
2. 扫描所有 `.md` 文档
3. 分析文档内容
4. **翻译英文文档为中文**
5. 生成中文分析报告

预期输出：

- `overview.md`: GSD 是一个 spec-driven development 系统
- `problems.md`: Context Rot 问题、Vibecoding 的不可靠性
- `solutions.md`: Wave 执行、200K 新鲜上下文、需求追踪
- `technical-points.md`: XML Prompt 格式、多 Agent 编排、原子提交
- `translated/`: 翻译后的原始文档

### 示例 2: 保留英文输出

```bash
/spec-analyzer https://github.com/xxx/yyy --lang en
```

**执行步骤**：
1. Clone 仓库
2. 分析文档（不翻译）
3. 生成英文分析报告

### 示例 3: Clone 并保留仓库

```bash
/spec-analyzer https://github.com/xxx/yyy --keep-repo
```

**执行步骤**：
1. Clone 仓库到 `/tmp/xxx/`
2. 分析文档
3. 生成报告
4. **保留临时仓库**（不删除，方便后续查看代码）

## 分析重点

在分析 spec-driven 仓库时，特别关注以下内容：

### 1. 问题域深度分析

- **Context Rot（上下文腐化）**: 随着对话进行，上下文质量下降的问题
- **Vibecoding 的局限**: 没有规范驱动的开发方式存在的问题
- **现有工具的不足**: 其他 spec-driven 工具的复杂性或不足

### 2. 解决方案关键点

- **Wave 执行**: 并行执行独立任务，依赖任务串行
- **新鲜上下文**: 每个任务在独立的 200K token 上下文中执行
- **需求追踪**: REQUIREMENTS.md 中的需求带 ID，可追踪到实现
- **验证机制**: 自动化测试和人工验证相结合

### 3. 技术要点提取

- **文件结构**: `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `PLAN.md` 等的作用
- **XML Prompt**: 结构化的任务描述格式
- **多 Agent 编排**: Orchestrator + 专门 Agent 的模式
- **原子提交**: 每个任务完成后立即提交

## 临时文件清理

### Clone 的仓库

- 默认位置：`/tmp/<repo-name>/`
- 清理策略：
  - **默认**：分析完成后自动删除
  - **`--keep-repo`**：保留仓库，用户手动清理

### 清理命令

```bash
# 手动清理所有临时分析的仓库
rm -rf /tmp/<repo-name>/

# 或者使用通配符清理（谨慎使用）
rm -rf /tmp/analyzed-*
```

## 常见问题

### Q1: Clone 失败怎么办？

**可能原因**：
- 网络问题（需要代理）
- 仓库不存在或私有
- Git 未安装

**解决方案**：
```bash
# 1. 设置代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 2. 如果是私有仓库，先手动 clone，然后使用本地路径
/spec-analyzer /path/to/local/repo

# 3. 完成后取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### Q2: 翻译质量不好怎么办？

**优化建议**：
- 使用 `--depth deep` 获取更多上下文
- 手动调整专业术语的翻译
- 参考 `translated/` 目录下的原文进行校对

### Q3: 如何只翻译不分析？

当前 skill 不支持仅翻译模式。如果只需要翻译：
```bash
# 手动 clone 并使用其他翻译工具
git clone https://github.com/xxx/yyy.git
# 然后使用专门的翻译工具
```

## 注意事项

1. **网络环境**：Clone GitHub 仓库可能需要代理
2. **磁盘空间**：大型仓库可能占用较多临时空间
3. **翻译准确性**：专业术语可能需要人工校对
4. **隐私仓库**：私有仓库需要先手动 clone，再使用本地路径分析
