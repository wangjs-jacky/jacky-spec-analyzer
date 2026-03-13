# optimize-skill 迭代优化系统实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 重写 optimize-skill SKILL.md，实现迭代优化系统，支持通过 Agent 调用 spec-analyzer 并自动评估和优化

**Architecture:** 采用单 Skill 内置循环 + 文件记录的混合方案。optimize-skill 作为主控制器，使用 Agent 工具启动子代理执行 spec-analyzer，通过 LLM 评估覆盖度和深度，自动迭代优化 SKILL.md

**Tech Stack:** Claude Code Skill (Markdown), Agent 工具, 文件系统

---

## Task 1: 备份现有 SKILL.md

**Files:**
- Create: `skills/optimize-skill/SKILL.md.backup`

**Step 1: 备份当前文件**

```bash
cp /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md.backup
```

**Step 2: 验证备份**

```bash
ls -la /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/
```

Expected: 看到 SKILL.md 和 SKILL.md.backup 两个文件

---

## Task 2: 重写 SKILL.md 元数据和概述

**Files:**
- Modify: `skills/optimize-skill/SKILL.md:1-30`

**Step 1: 更新元数据和概述**

替换文件开头部分为：

```markdown
---
name: optimize-skill
description: 迭代优化 Claude Code Skill 的质量。当用户想要改进 skill 的提示词、通过目标内容迭代优化 skill、自动评估和优化时触发。
---

# optimize-skill

迭代优化 Claude Code Skill 的质量，采用类似机器学习训练循环的方式：

1. 读取目标内容（期望输出）
2. 调用子代理执行目标 skill
3. LLM 评估输出质量
4. 生成优化建议
5. 应用优化到 skill 提示词
6. 重复直到达到评估阈值

## 核心特性

- **迭代优化循环**: 自动多轮优化，直到满足评估标准
- **子代理执行**: 使用 Agent 工具隔离执行目标 skill
- **LLM 评估**: 自动评估知识点覆盖度和内容深度
- **完整记录**: 保留每轮迭代的详细历史
```

**Step 2: 验证修改**

```bash
head -30 /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到新的元数据和概述

---

## Task 3: 添加使用方式和参数说明

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加到概述后)

**Step 1: 添加使用方式章节**

```markdown
## 使用方式

### 基础用法

```bash
# 迭代优化 spec-analyzer，指定目标内容和测试仓库
/optimize-skill spec-analyzer --target ./target.md --repo https://github.com/xxx/yyy

# 使用内联目标内容
/optimize-skill spec-analyzer --target "期望输出应包含：1. 问题域分析 2. 解决方案..." --repo https://github.com/xxx/yyy

# 指定评估阈值
/optimize-skill spec-analyzer --target ./target.md --repo https://github.com/xxx/yyy --threshold-coverage 95 --threshold-depth 4
```

### 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--target, -t` | 目标内容（文件路径或直接输入） | 必填 |
| `--repo, -r` | GitHub 仓库 URL | 必填 |
| `--threshold-coverage` | 覆盖度阈值 (0-100) | 90 |
| `--threshold-depth` | 深度阈值 (1-5) | 3.5 |
| `--max-iterations` | 最大迭代次数 | 5 |
| `--output, -o` | 优化记录输出目录 | `./optimization-records/` |
```

**Step 2: 验证修改**

```bash
grep -A 20 "## 使用方式" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到完整的使用方式章节

---

## Task 4: 添加工作流程说明

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加工作流程章节**

```markdown
## 工作流程

### 流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    迭代优化循环                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 读取目标内容 → 解析知识点清单                            │
│         ↓                                                    │
│  2. 初始化工作流目录                                         │
│         ↓                                                    │
│  3. Agent 调用目标 skill 提取内容                            │
│         ↓                                                    │
│  4. LLM 评估（覆盖度 + 深度）                                │
│         ↓                                                    │
│  5. 检查是否达到阈值 ──否──→ 生成优化建议                    │
│         │                          ↓                         │
│         │                    应用优化到 SKILL.md             │
│         │                          ↓                         │
│         │                    返回步骤 3                      │
│         ↓                                                    │
│        是                                                    │
│         ↓                                                    │
│  6. 输出最终报告                                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 步骤详解

#### 步骤 1: 读取目标内容

1. 读取 `--target` 指定的目标内容
2. 解析目标内容，提取知识点清单
3. 为每个知识点分配 ID、权重和类型

**知识点清单格式**：

| ID | 知识点 | 权重 | 类型 |
|----|--------|------|------|
| K1 | 问题描述 | 高 | 问题域 |
| K2 | 解决方案 | 高 | 解决方案 |
| K3 | 技术实现 | 中 | 技术要点 |

#### 步骤 2: 初始化工作流目录

创建优化记录目录：

```
./optimization-records/<timestamp>/
├── target.md                 # 目标内容
├── knowledge-points.json     # 知识点清单
├── repo-url.txt              # 分析的仓库 URL
└── iterations/               # 迭代记录
```

#### 步骤 3: 调用目标 Skill

使用 Agent 工具启动子代理：

```
使用 Agent 工具:
- subagent_type: "general-purpose"
- prompt: "执行 /spec-analyzer <repo> --depth deep"
- description: "执行 spec-analyzer 分析"
```

**重要**: 子代理在隔离环境中执行，不影响主会话上下文。

#### 步骤 4: LLM 评估

评估维度：

**1. 知识点覆盖度 (0-100%)**
- 完全覆盖：有详细解释
- 部分覆盖：有提及但不完整
- 未覆盖：完全没有提到

**2. 内容深度 (1-5 分)**
- 1: 仅提及
- 2: 简要说明
- 3: 详细解释
- 4: 包含示例
- 5: 深入分析

#### 步骤 5: 检查终止条件

终止条件：
- 覆盖度 >= `--threshold-coverage`
- 深度 >= `--threshold-depth`
- 或达到 `--max-iterations`

如果未达到阈值，生成优化建议并应用。

#### 步骤 6: 输出最终报告

生成包含以下内容的报告：
- 评估结果对比
- 迭代历史
- 主要优化点
- 未解决问题
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 工作流程" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到工作流程章节

---

## Task 5: 添加评估提示词模板

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加评估模板章节**

```markdown
## 评估提示词模板

在每轮迭代中，使用以下提示词进行评估：

```markdown
# 评估任务

## 目标知识点清单
{knowledge_points}

## 提取的内容
{extracted_content}

## 评估要求

请逐项评估每个目标知识点的覆盖情况：

### 1. 知识点覆盖度
对于每个目标知识点，判断：
- **完全覆盖**：有详细解释，包含关键细节
- **部分覆盖**：有提及但不完整
- **未覆盖**：完全没有提到

### 2. 内容深度评分 (1-5)
- 1: 仅提及名称
- 2: 简要说明（1-2 句话）
- 3: 详细解释（有完整描述）
- 4: 包含示例（有代码/图表/流程）
- 5: 深入分析（有对比、权衡、原理解释）

## 输出格式

请按以下 JSON 格式输出评估结果：

```json
{
  "coverage": {
    "total": <总数>,
    "covered": <完全覆盖数>,
    "partial": <部分覆盖数>,
    "missing": <未覆盖数>,
    "percentage": <覆盖百分比>
  },
  "depth": {
    "average": <平均深度分数>,
    "details": [
      {"id": "<知识点ID>", "score": <1-5>, "note": "<简短说明>"}
    ]
  },
  "missing_points": ["<未覆盖的知识点列表>"],
  "weak_points": ["<深度不足的知识点列表>"],
  "suggestions": ["<针对 SKILL.md 的具体修改建议>"]
}
```
```

**评估结果解读**：

| 覆盖度 | 深度 | 状态 |
|--------|------|------|
| >= 90% | >= 3.5 | ✅ 达标 |
| 70-89% | 2.5-3.4 | ⚠️ 需要优化 |
| < 70% | < 2.5 | ❌ 需要大幅改进 |
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 评估提示词模板" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到评估模板章节

---

## Task 6: 添加优化建议生成模板

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加优化模板章节**

```markdown
## 优化建议生成模板

当评估未达标时，使用以下模板生成优化建议：

```markdown
# 优化建议生成任务

## 当前评估结果
{evaluation_json}

## 当前 SKILL.md 内容
{current_skill_content}

## 任务

根据评估结果，生成具体的 SKILL.md 修改建议。

### 修改建议格式

对于每个问题，提供：

1. **问题描述**：具体是什么问题
2. **问题位置**：SKILL.md 中的哪个章节/步骤
3. **当前内容**：当前的提示词
4. **建议修改**：修改后的提示词
5. **修改原因**：为什么这样修改能解决问题

### 优化策略

| 问题类型 | 优化策略 |
|----------|----------|
| 未覆盖知识点 | 添加新的提取规则或检查清单 |
| 深度不足 | 细化提取指令，添加深度要求 |
| 信息遗漏 | 添加必检项和提示词引导 |
| 格式问题 | 调整输出模板 |
```

### 优化示例

**问题**: K4（文件体系）未被提取

**位置**: "扫描文档" 章节

**当前内容**:
> 递归查找所有 `.md` 文件，重点关注 README.md 和 docs/ 目录

**建议修改**:
> 递归查找所有 `.md` 文件，重点关注：
> - `README.md` - 项目介绍
> - `docs/` 目录 - 文档
> - `PROJECT.md` - 项目描述和目标
> - `REQUIREMENTS.md` - 需求清单
> - `ROADMAP.md` - 路线图
> - `STATE.md` - 状态追踪
> - 模板目录（如 `get-shit-done/templates/`）

**原因**: 原描述太笼统，没有指定 spec-driven 仓库特有的文件类型
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 优化建议生成模板" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到优化模板章节

---

## Task 7: 添加输出目录结构说明

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加输出结构章节**

```markdown
## 输出目录结构

优化完成后，生成以下文件结构：

```
./optimization-records/<timestamp>/
├── target.md                 # 目标内容
├── knowledge-points.json     # 解析后的知识点清单
├── repo-url.txt              # 分析的仓库 URL
├── iterations/
│   ├── 01/
│   │   ├── extracted.md      # 第 1 轮提取结果
│   │   ├── evaluation.json   # 第 1 轮评估
│   │   └── changes.md        # 第 1 轮修改建议
│   ├── 02/
│   │   └── ...
│   └── N/
├── final-report.md           # 最终优化报告
└── SKILL.md.final            # 优化后的最终版本
```

### 文件说明

| 文件 | 说明 |
|------|------|
| `target.md` | 用户提供的期望输出内容 |
| `knowledge-points.json` | 从目标内容解析的知识点清单 |
| `iterations/N/extracted.md` | 第 N 轮 skill 执行的输出 |
| `iterations/N/evaluation.json` | 第 N 轮的评估结果 |
| `iterations/N/changes.md` | 第 N 轮对 SKILL.md 的修改 |
| `final-report.md` | 完整的优化报告 |
| `SKILL.md.final` | 最终优化后的 skill 文件 |
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 输出目录结构" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到输出结构章节

---

## Task 8: 添加最终报告模板

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加报告模板章节**

```markdown
## 最终报告模板

优化完成后，生成 `final-report.md`：

```markdown
# {skill-name} 优化报告

## 基本信息
- **优化日期**: {timestamp}
- **目标仓库**: {repo_url}
- **迭代次数**: {iterations}
- **最终状态**: {success/failed}

## 评估结果

### 初始状态
- 覆盖度: {initial_coverage}%
- 深度: {initial_depth}

### 最终状态
- 覆盖度: {final_coverage}%
- 深度: {final_depth}

### 改进幅度
- 覆盖度提升: +{coverage_improvement}%
- 深度提升: +{depth_improvement}

## 迭代历史

| 轮次 | 覆盖度 | 深度 | 主要修改 |
|------|--------|------|----------|
| 1 | 60% | 2.5 | 添加文件体系提取规则 |
| 2 | 75% | 3.0 | 细化工作流程描述 |
| 3 | 90% | 3.6 | 添加深度要求 |

## 主要优化点

1. {优化点1}
2. {优化点2}
3. {优化点3}

## 未解决的问题

- {问题1}（建议后续处理）
- {问题2}（需要更多测试案例验证）

## 后续建议

- {后续改进建议}
```
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 最终报告模板" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到报告模板章节

---

## Task 9: 添加完整示例

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加)

**Step 1: 添加示例章节**

```markdown
## 完整示例

### 示例 1: 优化 spec-analyzer

**目标内容** (`target.md`):

```markdown
## GSD 工作流核心知识点

### 一、AI 编程工作流的重要性
- 上下文过长问题
- 需求分散问题
- 主代理负载过重

### 二、GSD 概述
- 规范驱动的 AI 编程工作流
- 30+ 个命令

### 三、GSD 文件体系
- PROJECT.md
- REQUIREMENTS.md
- ROADMAP.md
- STATE.md

### 四、完整工作流程
1. 初始化项目
2. 需求讨论
3. 生成计划
4. 执行编码
5. 验收测试

...
```

**执行命令**:

```bash
/optimize-skill spec-analyzer \
  --target ./target.md \
  --repo https://github.com/gsd-build/get-shit-done \
  --threshold-coverage 90 \
  --threshold-depth 3.5 \
  --max-iterations 5
```

**执行过程**:

```
=== 迭代优化开始 ===
目标: spec-analyzer
仓库: https://github.com/gsd-build/get-shit-done
阈值: 覆盖度 90%, 深度 3.5

[迭代 1]
- 调用 spec-analyzer...
- 评估: 覆盖度 65%, 深度 2.8
- 未达标，生成优化建议...
- 应用修改: 添加文件体系提取规则

[迭代 2]
- 调用 spec-analyzer...
- 评估: 覆盖度 78%, 深度 3.2
- 未达标，生成优化建议...
- 应用修改: 细化工作流程描述

[迭代 3]
- 调用 spec-analyzer...
- 评估: 覆盖度 92%, 深度 3.7
- 达标！

=== 优化完成 ===
最终报告: ./optimization-records/20260313-143022/final-report.md
优化后 Skill: ./optimization-records/20260313-143022/SKILL.md.final
```
```

**Step 2: 验证修改**

```bash
grep -A 5 "## 完整示例" /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 看到示例章节

---

## Task 10: 添加注意事项和清理旧内容

**Files:**
- Modify: `skills/optimize-skill/SKILL.md` (追加 + 清理)

**Step 1: 添加注意事项章节**

```markdown
## 注意事项

### 1. 迭代控制
- 一次优化聚焦 1-2 个核心问题
- 设置合理的最大迭代次数，避免无限循环
- 每轮迭代后检查进展，如果连续无改进可提前终止

### 2. 评估稳定性
- LLM 评估可能有波动，设置容差范围（如 ±5%）
- 关键优化点建议人工复核
- 深度评分相对主观，关注趋势而非绝对值

### 3. 文件管理
- 原始 SKILL.md 会自动备份
- 每轮迭代保留完整记录
- 可随时从 `SKILL.md.final` 恢复

### 4. 上下文管理
- 使用 Agent 工具隔离子代理执行
- 大型仓库可能导致提取内容过长
- 建议先在小型仓库测试

### 5. 代理配置
- Clone GitHub 仓库可能需要配置代理
- 参考 spec-analyzer 中的代理配置说明

## 常见问题

### Q1: 迭代不收敛怎么办？

**可能原因**：
- 目标内容与仓库内容不匹配
- 评估标准过高
- skill 本身设计有根本性问题

**解决方案**：
- 检查目标内容是否可从仓库中提取
- 适当降低阈值
- 分析未覆盖知识点的根本原因

### Q2: Agent 调用失败怎么办？

**可能原因**：
- 网络问题
- 仓库访问权限
- 上下文过长

**解决方案**：
- 检查网络和代理配置
- 确认仓库是公开的
- 尝试 smaller 测试仓库

### Q3: 如何只做单次评估不做迭代？

```bash
# 设置 max-iterations 为 1
/optimize-skill spec-analyzer --target ./target.md --repo https://github.com/xxx/yyy --max-iterations 1
```
```

**Step 2: 删除旧的重复内容**

删除文件中原有的旧内容（使用方式、工作流程、优化模式等章节），保留新的结构化内容。

**Step 3: 验证最终文件**

```bash
wc -l /Users/jacky/jacky-github/jacky-spec-analyzer/skills/optimize-skill/SKILL.md
```

Expected: 文件行数合理（约 300-400 行）

---

## Task 11: 提交更改

**Files:**
- All modified files

**Step 1: 查看更改**

```bash
cd /Users/jacky/jacky-github/jacky-spec-analyzer && git status
```

**Step 2: 添加文件**

```bash
git add skills/optimize-skill/SKILL.md skills/optimize-skill/SKILL.md.backup docs/plans/
```

**Step 3: 提交**

```bash
git commit -m "$(cat <<'EOF'
feat: 重构 optimize-skill 为迭代优化系统

- 添加迭代优化循环（类似机器学习训练）
- 使用 Agent 工具调用子代理执行目标 skill
- 添加 LLM 评估模块（覆盖度 + 深度）
- 添加优化建议生成模板
- 添加完整输出记录和最终报告
- 保留原始 SKILL.md 备份

设计文档: docs/plans/2026-03-13-iterative-optimization-design.md

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

**Step 4: 验证提交**

```bash
git log -1 --stat
```

Expected: 看到提交记录

---

## 验证清单

- [ ] SKILL.md 包含新的元数据（name, description）
- [ ] SKILL.md 包含使用方式和参数说明
- [ ] SKILL.md 包含完整工作流程图
- [ ] SKILL.md 包含评估提示词模板
- [ ] SKILL.md 包含优化建议生成模板
- [ ] SKILL.md 包含输出目录结构说明
- [ ] SKILL.md 包含最终报告模板
- [ ] SKILL.md 包含完整示例
- [ ] SKILL.md 包含注意事项和常见问题
- [ ] 原始文件已备份
- [ ] 提交记录完整
