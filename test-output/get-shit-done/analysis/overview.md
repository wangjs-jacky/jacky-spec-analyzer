# GSD (Get Shit Done) 概览

## 一句话简介

GSD 是一个轻量级的元提示和上下文工程系统，让 Claude Code 变得可靠。

## 核心价值

**解决 Context Rot（上下文腐化）问题** —— 随着 Claude 对话进行，上下文窗口被填充，导致输出质量下降。GSD 通过上下文工程、多代理编排和原子提交来保持一致的输出质量。

## 目标用户

- 独立开发者
- 想要描述需求并正确构建的人
- 不想玩"企业剧场"的小团队

## 快速开始

```bash
npx get-shit-done-cc@latest
```

验证：`/gsd:help`

## 核心工作流

```
/gsd:new-project     → 初始化项目
/gsd:discuss-phase N → 讨论阶段偏好
/gsd:plan-phase N    → 研究并规划
/gsd:execute-phase N → 执行计划
/gsd:verify-work N   → 验证工作
/gsd:complete-milestone → 完成里程碑
```

## 核心文件

| 文件 | 作用 |
|------|------|
| `PROJECT.md` | 项目愿景，始终加载 |
| `REQUIREMENTS.md` | 范围化的 v1/v2 需求 |
| `ROADMAP.md` | 阶段分解和状态跟踪 |
| `STATE.md` | 决定、阻碍、会话记忆 |
| `PLAN.md` | 带 XML 结构的原子任务 |
| `SUMMARY.md` | 执行结果和变更历史 |
