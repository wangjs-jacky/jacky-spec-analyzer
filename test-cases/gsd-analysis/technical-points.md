# GSD 技术要点

## 1. 核心概念

| 概念 | 定义 | 用途 |
|------|------|------|
| **Context Rot** | 上下文腐化，长对话中质量下降 | 理解问题本质 |
| **Wave** | 执行波次，独立任务并行 | 提高执行效率 |
| **Fresh Context** | 每个计划用独立的 200K 上下文 | 避免上下文污染 |
| **Atomic Commit** | 每个任务完成立即提交 | 可追溯、可回滚 |
| **Must-Haves** | 目标反向验证标准 | 确保目标达成 |
| **XML Prompt** | 结构化任务描述格式 | 精确无歧义 |

## 2. 关键实现

### 2.1 上下文工程

**目的**: 确保 Claude 始终有高质量上下文

**原理**: 通过多个专用文件管理不同类型的上下文

| 文件 | 作用 | 大小限制 |
|------|------|----------|
| `PROJECT.md` | 项目愿景，始终加载 | 保持简洁 |
| `research/` | 技术栈、特性、架构、陷阱 | 按需加载 |
| `REQUIREMENTS.md` | 需求列表，带 ID 追踪 | - |
| `ROADMAP.md` | 路线图，阶段划分 | - |
| `STATE.md` | 决策、阻塞、位置 | 跨会话记忆 |
| `PLAN.md` | 原子任务，XML 结构 | - |
| `SUMMARY.md` | 完成记录 | Git 历史 |

**示例 - PROJECT.md 结构**:
```markdown
# [Project Name]

## What This Is
[2-3 句描述]

## Core Value
[最重要的一件事]

## Requirements
### Validated - 已验证
### Active - 进行中
### Out of Scope - 明确排除

## Key Decisions
| Decision | Rationale | Outcome |
```

### 2.2 XML Prompt 格式

**目的**: 提供结构化、精确的任务描述

**原理**: 使用 XML 标签组织任务信息

**代码示例**:
```xml
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/app/api/auth/login/route.ts</files>
  <read_first>src/lib/auth.ts, src/middleware/auth.ts</read_first>
  <action>
    Use jose for JWT (not jsonwebtoken - CommonJS issues).
    Validate credentials against users table.
    Return httpOnly cookie on success.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200 + Set-Cookie</verify>
  <acceptance_criteria>
    - file.ext contains 'exact string'
    - output uses 'expected-value', NOT 'wrong-value'
  </acceptance_criteria>
  <done>Valid credentials return cookie, invalid return 401</done>
</task>
```

**关键字段**:
- `type`: `auto` | `checkpoint:human-verify` | `checkpoint:decision`
- `files`: 要修改的文件
- `read_first`: 执行前必须读取的文件
- `action`: 具体实现指令（包含具体值，不要模糊描述）
- `verify`: 验证命令
- `acceptance_criteria`: 可 grep 验证的条件

### 2.3 需求追踪系统

**目的**: 确保每个需求都被实现和验证

**原理**: 需求带 ID，映射到阶段，追踪状态

**代码示例**:
```markdown
# Requirements: [Project Name]

## v1 Requirements

### Authentication
- [ ] **AUTH-01**: User can sign up with email
- [ ] **AUTH-02**: User receives email verification

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Pending |
| AUTH-02 | Phase 1 | Complete |
```

### 2.4 Must-Haves 验证

**目的**: 任务完成 ≠ 目标达成，需要额外验证

**原理**: 在计划时定义"必须为真"的条件，执行后验证

**代码示例**:
```yaml
must_haves:
  truths:
    - "User can see existing messages"
    - "User can send a message"
  artifacts:
    - path: "src/components/Chat.tsx"
      provides: "Message list rendering"
      min_lines: 30
  key_links:
    - from: "src/components/Chat.tsx"
      to: "/api/chat"
      via: "fetch in useEffect"
```

### 2.5 多 Agent 编排

**目的**: 隔离上下文，提高执行质量

**原理**: Orchestrator 不做重活，只协调专门 Agent

| 阶段 | Orchestrator 职责 | Agent 职责 |
|------|-------------------|------------|
| 研究 | 协调、呈现结果 | 4 个并行研究 Agent |
| 计划 | 验证、管理迭代 | Planner + Checker |
| 执行 | 分组 Wave、追踪进度 | Executor 在独立上下文执行 |
| 验证 | 呈现结果、路由下一步 | Verifier + Debugger |

**结果**: 主上下文保持在 30-40%，工作在子 Agent 中完成

## 3. 数据流

```
用户描述需求
     ↓
/gsd:new-project
     ↓
PROJECT.md + REQUIREMENTS.md + ROADMAP.md
     ↓
/gsd:discuss-phase → CONTEXT.md
     ↓
/gsd:plan-phase → RESEARCH.md + PLAN.md (多个)
     ↓
/gsd:execute-phase
     ↓
[Wave 1] Plan 01 (200K context) → Commit
        Plan 02 (200K context) → Commit
     ↓
[Wave 2] Plan 03 (200K context) → Commit
     ↓
SUMMARY.md + VERIFICATION.md
     ↓
/gsd:verify-work → UAT.md
     ↓
用户确认 → 下一阶段
```

## 4. 扩展点

### 4.1 自定义 Agent

可以添加专门的 Agent 处理特定任务：
- `agents/gsd-*.md` 定义 Agent 行为

### 4.2 自定义命令

可以添加自定义命令：
- `commands/gsd/*.md` 定义命令行为

### 4.3 自定义模板

可以修改输出模板：
- `get-shit-done/templates/*.md`

### 4.4 配置选项

通过 `.planning/config.json` 配置：
- `mode`: yolo / interactive
- `granularity`: coarse / standard / fine
- `workflow.research`: true / false
- `workflow.plan_check`: true / false
- `git.branching_strategy`: none / phase / milestone

## 5. 关键文件结构

```
.planning/
├── PROJECT.md           # 项目愿景
├── REQUIREMENTS.md      # 需求（带 ID）
├── ROADMAP.md           # 路线图
├── STATE.md             # 当前状态
├── config.json          # 配置
├── research/            # 研究结果
│   ├── STACK.md
│   ├── FEATURES.md
│   ├── ARCHITECTURE.md
│   └── PITFALLS.md
└── phases/
    └── 01-foundation/
        ├── 01-CONTEXT.md
        ├── 01-RESEARCH.md
        ├── 01-01-PLAN.md
        ├── 01-01-SUMMARY.md
        ├── 01-02-PLAN.md
        └── 01-VERIFICATION.md
```

---

*分析日期: 2026-03-13*
