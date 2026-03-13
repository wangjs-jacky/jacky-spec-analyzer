# CLAUDE.md - jacky-spec-analyzer 项目说明

## 项目目的

**jacky-spec-analyzer** 是一个 Claude Code Skills 开发项目，目的是生成高质量的 **spec-analyzer** 和 **optimize-skill** 两个 skills。

## 核心 Skills

### 1. spec-analyzer

**用途**: 分析 spec-driven 仓库，提取核心知识

**功能**:
- 分析项目要解决的问题（问题域）
- 提取解决方案和工作流程
- 提取关键技术要点
- 输出 Markdown 和可交互 HTML

**使用方式**:
```bash
/spec-analyzer https://github.com/xxx/yyy --depth deep
```

### 2. optimize-skill

**用途**: 系统化优化 Claude Code Skill 的质量

**功能**:
- 识别 skill 的问题和改进点
- 生成具体的优化建议
- 验证优化效果

**使用方式**:
```bash
/optimize-skill <skill-name> --test-case <repo-url>
```

## 开发流程

本项目使用 **迭代优化** 的方式开发 skills：

```
┌─────────────────────────────────────────────────────────────┐
│                   Skill 开发迭代流程                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 编写 SKILL.md 骨架                                       │
│         ↓                                                    │
│  2. 选择测试案例仓库                                         │
│         ↓                                                    │
│  3. 执行 spec-analyzer 分析                                  │
│         ↓                                                    │
│  4. 生成分析报告 + 优化建议                                   │
│         ↓                                                    │
│  5. 应用优化到 SKILL.md                                      │
│         ↓                                                    │
│  6. 使用新测试案例验证                                        │
│         ↓                                                    │
│  7. 重复 3-6 直到满意                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 项目结构

```
jacky-spec-analyzer/
├── CLAUDE.md                    # 本文件 - 项目说明
├── README.md                    # 项目介绍
├── skills/
│   ├── spec-analyzer/
│   │   └── SKILL.md             # 核心分析 skill
│   └── optimize-skill/
│       └── SKILL.md             # Skill 优化 skill
├── workflows/
│   └── optimize-analyzer.md     # 优化工作流（手动）
├── templates/
│   ├── analysis-template.md     # 分析模板
│   └── html/                    # HTML 输出模板
├── test-cases/
│   └── gsd/                     # GSD 仓库（测试案例）
└── test-cases/
    └── gsd-analysis/            # GSD 分析结果
        ├── overview.md
        ├── problems.md
        ├── solutions.md
        ├── technical-points.md
        └── optimization-report.md
```

## 测试案例

### 已分析

| 仓库 | 分析日期 | 状态 | 报告位置 |
|------|----------|------|----------|
| [GSD](https://github.com/gsd-build/get-shit-done) | 2026-03-13 | ✅ 完成 | `test-cases/gsd-analysis/` |

### 待分析

| 仓库 | 用途 | 优先级 |
|------|------|--------|
| [vercel-labs/skills](https://github.com/vercel-labs/skills) | 验证优化效果 | 高 |
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | Claude Code 官方 | 中 |

## 当前迭代状态

**迭代**: 1
**当前测试案例**: GSD
**优化建议状态**: 待应用

### 待应用的优化

1. [ ] 细化技术要点提取规则
2. [ ] 添加 spec-driven 特征识别
3. [ ] 改进数据流描述模板
4. [ ] 添加模板文件处理说明

## 开发命令

```bash
# 链接 skills 到全局
j-skills link skills/spec-analyzer
j-skills link skills/optimize-skill

# 验证链接
j-skills link --list

# 运行分析（在新会话中）
/spec-analyzer https://github.com/xxx/yyy --depth deep
```

## 注意事项

1. **迭代优化**: 每次 skill 改进后，需要用新测试案例验证
2. **保留历史**: 测试案例的分析结果保存在 `test-cases/` 目录
3. **文档同步**: SKILL.md 更新后，同步更新 `templates/analysis-template.md`

## 参考资料

- [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) - spec-driven 开发系统参考
- [vercel-labs/skills](https://github.com/vercel-labs/skills) - Skill 开发最佳实践
- [jacky-skills](/Users/jacky/jacky-github/jacky-skills) - 本地 skills 仓库

---

*最后更新: 2026-03-13*
