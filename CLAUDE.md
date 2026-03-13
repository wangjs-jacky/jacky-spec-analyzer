# jacky-spec-analyzer

开发高质量 Claude Code Skills 的项目。

## 核心 Skills

### spec-analyzer

分析 spec-driven 仓库，提取核心知识：问题域、解决方案、技术要点。

```bash
/spec-analyzer https://github.com/xxx/yyy --depth deep
```

### optimize-skill

迭代优化 Claude Code Skill 质量。

```bash
/optimize-skill spec-analyzer --target ./target.md --repo https://github.com/xxx/yyy
```

## 项目结构

```
skills/
├── spec-analyzer/SKILL.md    # 分析 skill
└── optimize-skill/SKILL.md   # 优化 skill
test-cases/                    # 测试仓库（GSD）
docs/plans/                    # 设计文档
```

## 使用

```bash
# 链接 skills
j-skills link skills/spec-analyzer
j-skills link skills/optimize-skill
```
