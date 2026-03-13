# spec-analyzer Skill 设计文档

> 创建日期: 2026-03-13
> 状态: 已批准

## 概述

**项目名称**: jacky-spec-analyzer
**目的**: 分析 spec-driven 仓库，提取核心知识并生成有条理的文档

## 核心需求

### 用户场景

分析类似 GSD (Get Shit Done) 这样的 spec-driven 仓库，提取以下信息：

1. **问题域**
   - 项目要解决什么问题？（如 Context Rot 上下文腐化）
   - 没有这套流程之前的开发范式存在什么问题？

2. **解决方案**
   - 这套系统如何解决这些问题？
   - 核心工作流程是什么？

3. **技术要点**
   - 关键实现细节（如 Wave 并行执行、200K 新鲜上下文、需求 ID 追踪等）
   - 每个特性的具体作用

### 输入输出

**输入**:
- 本地仓库路径
- 远程 GitHub URL

**输出**:
- Markdown 文档（适合 Obsidian）
- 可交互 HTML（用于 GitHub Pages）

## 技术设计

### 仓库结构

```
jacky-spec-analyzer/
├── skills/
│   └── spec-analyzer/
│       └── SKILL.md          # 核心分析 skill
├── workflows/
│   └── optimize-analyzer.md  # 优化工作流
├── templates/
│   └── analysis-template.md  # 输出模板
├── examples/
│   └── gsd-analysis/         # GSD 分析示例
└── README.md
```

### spec-analyzer Skill

**触发方式**: `/spec-analyzer <repo-url-or-path> [options]`

**参数**:
- `--output, -o`: 输出目录（默认 `./analyzed/`）
- `--depth, -d`: 分析深度（quick/deep）
- `--lang, -l`: 输出语言（zh-CN/en）

**输出结构**:
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

### optimize-analyzer 工作流

**目的**: 迭代优化 spec-analyzer 的输出质量

**流程**:
1. 运行 `spec-analyzer` 分析目标仓库
2. 人工审查输出，标注需要改进的地方
3. 更新 `analysis-template.md` 模板
4. 更新 `SKILL.md` 的分析指令
5. 重新运行验证

## 实现计划

### Phase 1: 基础结构

1. 创建仓库和目录结构
2. 创建 spec-analyzer skill 骨架
3. 实现基本的文档扫描功能

### Phase 2: 核心分析

1. 实现问题域提取逻辑
2. 实现解决方案提取逻辑
3. 实现技术要点提取逻辑

### Phase 3: 输出生成

1. 实现 Markdown 输出
2. 实现 HTML 输出（用于 GitHub Pages）
3. 实现结构化 JSON 输出

### Phase 4: 优化工作流

1. 创建优化工作流文档
2. 创建输出模板
3. 使用 GSD 作为测试案例迭代优化

## 参考项目

- GSD (Get Shit Done): https://github.com/gsd-build/get-shit-done
