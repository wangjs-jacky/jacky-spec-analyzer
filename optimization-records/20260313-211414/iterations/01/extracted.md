# GSD (Get Shit Done) 仓库完整分析报告

## 一、项目概览

### 1.1 一句话简介

**GSD 是一个轻量级但功能强大的元提示、上下文工程和规范驱动开发系统，用于 Claude Code、OpenCode、Gemini CLI 和 Codex，解决了上下文腐化（Context Rot）问题。**

### 1.2 核心价值

GSD 解决了 AI 编程工具（如 Claude Code）在长时间对话中质量下降的问题。通过精心设计的上下文管理、多 Agent 编排和原子提交机制，确保每个任务都在新鲜的 200K token 上下文中执行，实现一致的代码质量。

### 1.3 目标用户

- 想要描述需求就能正确构建功能的个人开发者
- 不想玩"企业角色扮演游戏"（如冲刺会议、故事点、Jira 工作流）的创意人士
- 信任 AI 编程工具但需要可靠性的工程师

### 1.4 快速开始

```bash
# 安装
npx get-shit-done-cc@latest

# 验证
/gsd:help

# 推荐运行模式
claude --dangerously-skip-permissions
```

---

## 二、问题分析

### 2.1 核心问题

> **AI 编程工具随着对话进行，上下文质量下降，导致代码质量不可预测**

### 2.2 背景与现状

#### 之前的做法

在没有 GSD 之前，用户使用 AI 编程工具的方式：

1. **Vibecoding**：描述想要什么，AI 生成代码，期望结果正确
2. **长对话模式**：在同一个会话中持续添加需求，让 AI 不断修改代码
3. **手动管理**：用户自己尝试管理上下文，手动清理对话历史

#### 现有方案的痛点

| 痛点 | 描述 | 根本原因 | 影响 |
|------|------|----------|------|
| **Context Rot（上下文腐化）** | 随着对话进行，AI 输出质量下降 | 上下文窗口被无关信息填满，关键信息被稀释 | 代码质量不可预测，后期代码越来越差 |
| **Vibecoding 不可靠** | 描述需求后生成的代码不一致或有问题 | 缺乏规范化的需求捕获和验证机制 | 需要大量人工调试和修正 |
| **缺乏系统性追踪** | 需求分散在对话中，难以追踪完成状态 | 没有结构化的需求文档和状态管理 | 不知道什么已完成、什么待做 |
| **其他 Spec-driven 工具过于复杂** | BMAD、Speckit 等工具引入了企业级流程 | 假设用户是大团队，需要复杂流程 | 个人开发者不想玩"企业角色扮演" |
| **上下文过长导致信息丢失** | 重要决策和约束被埋在大量对话中 | 上下文窗口有限，信息密度降低 | AI 忘记之前的决策，产生矛盾代码 |

### 2.3 问题根源分析

1. **上下文窗口限制**：Claude 有 200K token 的上下文限制，但质量在接近上限前就开始下降
2. **信息稀释**：随着对话进行，关键信息（需求、约束、决策）被大量生成的代码和调试信息稀释
3. **缺乏结构化**：没有系统化的方式来捕获、组织和追踪需求
4. **单一 Agent 负担过重**：一个 Agent 试图完成所有工作（研究、规划、执行、验证），导致质量下降

---

## 三、解决方案

### 3.1 核心思路

> **将复杂任务分解为小任务，每个任务在独立的 200K 新鲜上下文中执行，通过结构化文档追踪需求和状态**

GSD 的核心设计理念：
1. **上下文工程**：精心管理每个 Agent 能看到的信息
2. **多 Agent 编排**：专门的 Agent 做专门的事
3. **原子提交**：每个任务完成后立即提交，保持清晰的 Git 历史
4. **规范驱动**：通过结构化文档（PROJECT.md、REQUIREMENTS.md 等）捕获需求

### 3.2 文件体系

GSD 创建了一组结构化文档来管理项目：

| 文件 | 作用 | 内容 |
|------|------|------|
| `PROJECT.md` | 项目愿景 | 项目背景、目标、技术栈、约束 |
| `REQUIREMENTS.md` | 需求追踪 | 带 ID 的需求列表，可追踪到实现 |
| `ROADMAP.md` | 阶段规划 | Phase 分解和状态追踪 |
| `STATE.md` | 会话记忆 | 决策、阻塞点、当前位置 |
| `CONTEXT.md` | 实现偏好 | 用户对特定 Phase 的实现偏好 |
| `RESEARCH.md` | 领域研究 | 技术调研结果 |
| `PLAN.md` | 执行计划 | XML 结构的原子任务 |
| `SUMMARY.md` | 执行结果 | 完成的任务和决策 |
| `VERIFICATION.md` | 验证报告 | 目标达成情况检查 |

### 3.3 完整工作流程

GSD 定义了一个清晰的 5 阶段工作流：

```
┌────────────────────────────────────────────────────────────────┐
│                    GSD 完整工作流程                              │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. /gsd:new-project     →  初始化项目                          │
│     ├── Questions        →  深度需求收集                        │
│     ├── Research         →  并行领域研究（可选）                 │
│     ├── Requirements     →  提取 v1/v2 需求                     │
│     └── Roadmap          →  创建 Phase 分解                     │
│                                                                 │
│  2. /gsd:discuss-phase   →  捕获实现偏好                        │
│     └── CONTEXT.md       →  锁定用户的实现选择                   │
│                                                                 │
│  3. /gsd:plan-phase      →  创建执行计划                        │
│     ├── Research         →  技术调研（如需要）                   │
│     ├── Plan             →  创建 2-3 个原子任务                  │
│     └── Verify           →  验证计划可达目标                     │
│                                                                 │
│  4. /gsd:execute-phase   →  并行执行                            │
│     ├── Wave 分组        →  依赖分析后分组执行                   │
│     ├── Fresh Context    →  每个 Agent 200K 新鲜上下文          │
│     ├── Atomic Commit    →  每个任务独立提交                     │
│     └── Verify           →  检查目标达成                         │
│                                                                 │
│  5. /gsd:verify-work     →  用户验收测试                        │
│     ├── UAT              →  逐项用户测试                         │
│     ├── Diagnose         →  自动诊断问题                         │
│     └── Fix Plans        →  创建修复计划                         │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  循环：discuss → plan → execute → verify（直到 milestone 完成）  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  /gsd:complete-milestone →  归档 milestone，打 release tag      │
│  /gsd:new-milestone      →  开始下一个版本                       │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 3.4 关键设计决策

| 决策 | 原因 | 权衡 |
|------|------|------|
| **每个任务独立 Agent** | 避免上下文腐化，保证质量 | 更多 token 消耗，但质量稳定 |
| **Wave 并行执行** | 加速执行，独立任务同时进行 | 需要依赖分析，复杂度增加 |
| **XML 格式计划** | 结构化描述，Claude 更好理解 | 可读性稍差，但执行更可靠 |
| **原子提交** | 每个任务独立提交，可回滚 | 提交数量多，但历史清晰 |
| **200K token 上下文** | Claude 的上限，保证足够空间 | 可能浪费，但避免质量下降 |

### 3.5 与现有方案对比

| 维度 | Vibecoding | 其他 Spec-driven 工具 | GSD |
|------|------------|----------------------|-----|
| **上下文管理** | 无，质量下降 | 有，但复杂 | 自动管理，用户无感知 |
| **需求追踪** | 对话中分散 | 有，但需要手动维护 | 自动生成 REQUIREMENTS.md |
| **工作流复杂度** | 简单但不可靠 | 复杂，企业级 | 简单，几个命令 |
| **质量保证** | 依赖运气 | 依赖流程 | 自动验证 + 用户验收 |
| **适合场景** | 快速原型 | 大团队 | 个人开发者/小团队 |

---

## 四、技术要点

### 4.1 核心概念

| 概念 | 定义 | 用途 |
|------|------|------|
| **Phase** | 一个开发阶段，对应 ROADMAP.md 中的一项 | 组织开发工作，粒度可控 |
| **Wave** | 执行阶段内的并行组，依赖分析后确定 | 最大化并行执行 |
| **Plan** | 具体执行计划，包含 2-3 个原子任务 | 指导 Agent 执行 |
| **Agent** | 专门化的 AI 代理（研究、规划、执行、验证） | 分工明确，质量稳定 |
| **Milestone** | 一个版本发布，包含多个 Phase | 组织版本迭代 |

### 4.2 Wave 执行机制

Wave 是 GSD 的核心创新之一，实现了依赖感知的并行执行：

```
┌─────────────────────────────────────────────────────────────────────┐
│  PHASE EXECUTION                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WAVE 1 (parallel)          WAVE 2 (parallel)          WAVE 3       │
│  ┌─────────┐ ┌─────────┐    ┌─────────┐ ┌─────────┐    ┌─────────┐ │
│  │ Plan 01 │ │ Plan 02 │ →  │ Plan 03 │ │ Plan 04 │ →  │ Plan 05 │ │
│  │         │ │         │    │         │ │         │    │         │ │
│  │ User    │ │ Product │    │ Orders  │ │ Cart    │    │ Checkout│ │
│  │ Model   │ │ Model   │    │ API     │ │ API     │    │ UI      │ │
│  └─────────┘ └─────────┘    └─────────┘ └─────────┘    └─────────┘ │
│       │           │              ↑           ↑              ↑       │
│       └───────────┴──────────────┴───────────┘              │       │
│              Dependencies: Plan 03 needs Plan 01            │       │
│                          Plan 04 needs Plan 02              │       │
│                          Plan 05 needs Plans 03 + 04        │       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Wave 分组规则：**
- **独立任务 → 同一 Wave**：无依赖关系的任务并行执行
- **依赖任务 → 后续 Wave**：等待依赖完成后执行
- **文件冲突 → 串行或同 Plan**：避免同时修改同一文件

### 4.3 XML Prompt 格式

每个 Plan 使用 XML 结构描述任务，Claude 更容易准确执行：

```xml
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    Use jose for JWT (not jsonwebtoken - CommonJS issues).
    Validate credentials against users table.
    Return httpOnly cookie on success.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200 + Set-Cookie</verify>
  <done>Valid credentials return cookie, invalid return 401</done>
</task>
```

**关键元素：**
- `<name>`：任务名称
- `<files>`：涉及的文件
- `<action>`：具体操作
- `<verify>`：验证命令
- `<done>`：完成标准

### 4.4 多 Agent 编排架构

GSD 使用 Orchestrator + 专门 Agent 的模式：

```
┌──────────────────────────────────────────────────────────────┐
│                     ORCHESTRATOR (轻量)                       │
│                  协调、收集、路由结果                          │
│                  上下文占用：~10-15%                          │
└────────────────────┬─────────────────────────────────────────┘
                     │
         ┌───────────┼───────────┬───────────────┐
         │           │           │               │
         ▼           ▼           ▼               ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ Research │ │ Planner  │ │ Executor │ │ Verifier │
   │  Agent   │ │  Agent   │ │  Agent   │ │  Agent   │
   │          │ │          │ │          │ │          │
   │ 200K     │ │ 200K     │ │ 200K     │ │ 200K     │
   │ fresh    │ │ fresh    │ │ fresh    │ │ fresh    │
   │ context  │ │ context  │ │ context  │ │ context  │
   └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

**Agent 职责分工：**

| Agent | 职责 | 输入 | 输出 |
|-------|------|------|------|
| **gsd-phase-researcher** | 技术领域调研 | Phase 描述、CONTEXT.md | RESEARCH.md |
| **gsd-planner** | 创建执行计划 | RESEARCH.md、CONTEXT.md | PLAN.md |
| **gsd-executor** | 执行计划 | PLAN.md | 代码 + SUMMARY.md |
| **gsd-verifier** | 验证目标达成 | PLAN.md + 代码 | VERIFICATION.md |
| **gsd-plan-checker** | 验证计划质量 | PLAN.md | 通过/修改建议 |

### 4.5 原子提交机制

每个任务完成后立即提交，保持清晰的 Git 历史：

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
lmn012o feat(08-02): create registration endpoint
```

**提交类型：**
- `feat`：新功能
- `fix`：Bug 修复
- `test`：测试相关
- `refactor`：代码重构
- `chore`：配置/工具
- `docs`：文档

### 4.6 偏差处理规则

执行过程中发现计划外工作时，Agent 自动应用以下规则：

| 规则 | 触发条件 | 处理方式 |
|------|----------|----------|
| **Rule 1: Auto-fix bugs** | 代码不按预期工作 | 自动修复，记录到 SUMMARY |
| **Rule 2: Auto-add missing critical** | 缺少关键功能（安全、正确性） | 自动添加，记录到 SUMMARY |
| **Rule 3: Auto-fix blocking issues** | 阻塞当前任务的问题 | 自动解决，记录到 SUMMARY |
| **Rule 4: Ask about architectural changes** | 需要重大结构修改 | 停止，询问用户 |

---

## 五、进阶功能

### 5.1 Milestone 管理

Milestone 是版本发布的组织单位，包含多个 Phase：

```bash
# 开始新 milestone
/gsd:new-milestone v2.0

# 完成后归档
/gsd:audit-milestone    # 验证所有需求已完成
/gsd:complete-milestone # 归档并打 tag
```

**Milestone 生命周期：**
1. `/gsd:new-milestone` → 定义新版本需求
2. 循环执行 Phase 直到完成
3. `/gsd:audit-milestone` → 检查需求覆盖
4. `/gsd:complete-milestone` → 归档、打 tag

### 5.2 Quick 模式

对于不需要完整规划的小任务：

```bash
/gsd:quick
> "Add dark mode toggle to settings"
```

**Quick 模式特点：**
- 跳过研究、计划检查、验证
- 仍然有原子提交和状态追踪
- 结果存储在 `.planning/quick/`

**可选参数：**
- `--discuss`：执行前收集上下文
- `--full`：启用计划检查和验证

### 5.3 Brownfield 工作流

对于已有代码库：

```bash
# 分析现有代码
/gsd:map-codebase

# 然后初始化项目（问题会聚焦在新增内容）
/gsd:new-project
```

**map-codebase 输出：**
- `codebase/STACK.md`：技术栈分析
- `codebase/ARCHITECTURE.md`：架构分析
- `codebase/CONVENTIONS.md`：代码规范
- `codebase/CONCERNS.md`：关注点/风险点

### 5.4 上下文监控

GSD 提供上下文使用监控，在接近限制时警告：

| 级别 | 剩余上下文 | 行为 |
|------|-----------|------|
| Normal | > 35% | 无警告 |
| WARNING | <= 35% | 建议完成当前任务，避免开始新复杂工作 |
| CRITICAL | <= 25% | 立即停止，保存状态（`/gsd:pause-work`） |

### 5.5 配置选项

```json
{
  "mode": "interactive",        // interactive | yolo（自动批准）
  "granularity": "standard",    // coarse | standard | fine
  "model_profile": "balanced",  // quality | balanced | budget
  "workflow": {
    "research": true,           // 领域研究
    "plan_check": true,         // 计划验证
    "verifier": true,           // 执行验证
    "nyquist_validation": true  // 测试覆盖验证
  }
}
```

**Model Profile：**

| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

---

## 六、优缺点分析

### 6.1 优点

| 优点 | 说明 |
|------|------|
| **解决上下文腐化** | 每个 Agent 在新鲜上下文中执行，质量稳定 |
| **简化工作流** | 几个命令就能完成复杂项目，无需企业级流程 |
| **清晰的 Git 历史** | 原子提交让每次修改可追踪、可回滚 |
| **需求可追踪** | REQUIREMENTS.md 中每个需求有 ID，可追踪到实现 |
| **自动验证** | 自动检查目标达成，减少人工验证负担 |
| **灵活配置** | 可调整粒度、模型、验证级别等 |
| **多平台支持** | 支持 Claude Code、OpenCode、Gemini CLI、Codex |

### 6.2 潜在缺点

| 缺点 | 说明 | 缓解措施 |
|------|------|----------|
| **Token 消耗较高** | 多 Agent 编排消耗更多 token | 使用 budget profile，禁用可选 Agent |
| **学习曲线** | 需要理解 Phase/Wave/Plan 等概念 | 文档完善，命令直观 |
| **需要信任自动化** | 用户需要信任 Agent 的执行 | `/gsd:verify-work` 提供人工验收机会 |
| **依赖 Claude Code** | 主要针对 Claude Code 优化 | 也支持其他平台 |
| **不适合大团队** | 设计针对个人/小团队 | 大团队可能需要更复杂工具 |

### 6.3 适用场景

**推荐使用：**
- 个人开发者构建完整应用
- 小团队快速迭代
- 需要可靠 AI 编程的场景
- 复杂项目的规范化开发

**不推荐使用：**
- 大型企业团队（需要更复杂流程）
- 简单脚本（Quick 模式可能过度）
- 不信任 AI 自动化的场景

---

## 七、核心命令参考

### 7.1 核心工作流命令

| 命令 | 用途 | 使用时机 |
|------|------|----------|
| `/gsd:new-project` | 初始化项目 | 项目开始时 |
| `/gsd:discuss-phase [N]` | 捕获实现偏好 | 规划前 |
| `/gsd:plan-phase [N]` | 创建执行计划 | 执行前 |
| `/gsd:execute-phase <N>` | 执行计划 | 规划完成后 |
| `/gsd:verify-work [N]` | 用户验收测试 | 执行完成后 |

### 7.2 导航命令

| 命令 | 用途 |
|------|------|
| `/gsd:progress` | 查看当前位置和下一步 |
| `/gsd:resume-work` | 从上次会话恢复 |
| `/gsd:pause-work` | 保存状态以便暂停 |
| `/gsd:help` | 显示所有命令 |

### 7.3 Phase 管理命令

| 命令 | 用途 |
|------|------|
| `/gsd:add-phase` | 添加新 Phase |
| `/gsd:insert-phase [N]` | 在指定位置插入 Phase |
| `/gsd:remove-phase [N]` | 移除 Phase |

### 7.4 工具命令

| 命令 | 用途 |
|------|------|
| `/gsd:quick` | 快速任务 |
| `/gsd:debug [desc]` | 系统化调试 |
| `/gsd:settings` | 配置设置 |
| `/gsd:set-profile <profile>` | 切换模型配置 |

---

## 八、总结

GSD 是一个精心设计的 AI 编程辅助系统，通过以下核心机制解决了 AI 编程的可靠性问题：

1. **上下文工程**：每个任务在新鲜的 200K 上下文中执行，避免上下文腐化
2. **多 Agent 编排**：专门化的 Agent 分工明确，质量稳定
3. **Wave 并行执行**：依赖感知的并行执行，最大化效率
4. **原子提交**：每个任务独立提交，历史清晰可追踪
5. **规范驱动**：结构化文档捕获需求和状态

对于想要"描述需求就能正确构建功能"的个人开发者，GSD 提供了一个不需要"企业角色扮演"的可靠解决方案。
