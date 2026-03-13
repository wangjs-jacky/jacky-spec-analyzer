# GSD 技术要点

## 1. 核心概念

| 概念 | 定义 | 用途 |
|------|------|------|
| **Context Rot** | 上下文腐化，对话质量随时间下降 | 问题定义 |
| **Wave 执行** | 并行执行独立任务，依赖任务串行 | 加速开发 |
| **新鲜上下文** | 每个任务使用独立的 200K token 上下文 | 保证质量 |
| **原子提交** | 每个任务完成后立即 git commit | 可追溯性 |
| **Nyquist 验证** | 在执行前映射自动化测试覆盖 | 质量保证 |

## 2. 关键实现

### 2.1 XML 任务格式

**目的**: 为 Claude 提供结构化的任务指令

**原理**: XML 格式对 Claude 解析最优，包含明确的 action、verify、done 标准

```xml
<task type="auto">
  <name>创建登录端点</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    使用 jose 处理 JWT（不是 jsonwebtoken - CommonJS 问题）。
    根据 users 表验证凭据。
    成功时返回 httpOnly cookie。
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login 返回 200 + Set-Cookie</verify>
  <done>有效凭据返回 cookie，无效返回 401</done>
</task>
```

### 2.2 多代理编排

**目的**: 让主会话保持轻量

**原理**: 编排器启动专门代理，收集结果，路由到下一步

```
编排器模式:
┌─────────────────────────────────────────────────────────┐
│  主会话（编排器）                                         │
│  ├── 启动 Agent A → 等待 → 收集结果                      │
│  ├── 启动 Agent B → 等待 → 收集结果                      │
│  └── 整合结果 → 路由到下一步                             │
└─────────────────────────────────────────────────────────┘
```

**代理类型**:
- `gsd-planner`: 创建计划
- `gsd-executor`: 执行实现
- `gsd-verifier`: 验证结果
- `gsd-phase-researcher`: 研究阶段生态
- `gsd-debugger`: 调试问题

### 2.3 上下文工程

**目的**: 让 Claude 拥有项目级理解

**原理**: 通过结构化文档提供持久上下文

| 文件 | 加载时机 | 大小限制 |
|------|----------|----------|
| `PROJECT.md` | 始终 | 无限制 |
| `REQUIREMENTS.md` | 规划时 | 无限制 |
| `ROADMAP.md` | 规划时 | 无限制 |
| `STATE.md` | 恢复时 | 无限制 |
| `PLAN.md` | 执行时 | < 10KB |

### 2.4 Wave 执行算法

**目的**: 最大化并行，最小化等待

**原理**:
1. 分析计划依赖关系
2. 独立计划 → 同一波次 → 并行执行
3. 依赖计划 → 后续波次 → 等待依赖

```
依赖分析:
Plan 01 (User Model) ─────┐
Plan 02 (Product Model) ──┤
                          ├──→ Plan 03 (Orders API)
                          │    需要 Plan 01
                          │
                          └──→ Plan 04 (Cart API)
                               需要 Plan 02

Wave 1: [Plan 01, Plan 02] 并行
Wave 2: [Plan 03, Plan 04] 并行（等待 Wave 1 完成）
```

## 3. 数据流

```
用户输入
    │
    ▼
/gsd:new-project
    │
    ├──→ 提问 Agent → PROJECT.md
    ├──→ 研究 Agent → research/
    ├──→ 需求提取 → REQUIREMENTS.md
    └──→ 路线图 → ROADMAP.md
    │
    ▼
/gsd:discuss-phase N
    │
    └──→ CONTEXT.md
    │
    ▼
/gsd:plan-phase N
    │
    ├──→ 研究 Agent → RESEARCH.md
    ├──→ 规划 Agent → PLAN.md
    └──→ 检查 Agent → 验证通过
    │
    ▼
/gsd:execute-phase N
    │
    ├──→ Wave 1: [Executor A, Executor B] → commits
    ├──→ Wave 2: [Executor C] → commits
    └──→ 验证 Agent → VERIFICATION.md
    │
    ▼
/gsd:verify-work N
    │
    └──→ UAT.md + 修复计划（如需要）
```

## 4. 扩展点

### 4.1 自定义模型配置

```json
{
  "model_profile": "balanced",
  "agents": {
    "gsd-planner": "opus",
    "gsd-executor": "sonnet"
  }
}
```

### 4.2 Git 分支策略

```json
{
  "git": {
    "branching_strategy": "phase",
    "phase_branch_template": "gsd/phase-{phase}-{slug}"
  }
}
```

### 4.3 工作流开关

```json
{
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true
  }
}
```

## 5. 关键命令映射

| 阶段 | 命令 | 创建的文件 |
|------|------|-----------|
| 初始化 | `/gsd:new-project` | PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md |
| 讨论 | `/gsd:discuss-phase N` | CONTEXT.md |
| 规划 | `/gsd:plan-phase N` | RESEARCH.md, PLAN.md |
| 执行 | `/gsd:execute-phase N` | SUMMARY.md, VERIFICATION.md |
| 验证 | `/gsd:verify-work N` | UAT.md |
| 完成 | `/gsd:complete-milestone` | 归档到 MILESTONES.md |
