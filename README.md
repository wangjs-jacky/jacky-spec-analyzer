# jacky-spec-analyzer

分析 spec-driven 仓库，提取核心知识并生成有条理的文档。

## Skills

本项目包含两个核心 skills：

### 1. spec-analyzer

分析 spec-driven 仓库，提取核心信息：

- 分析项目要解决的问题
- 提取解决方案和工作流程
- 提取关键技术要点
- 输出 Markdown 和可交互 HTML

```bash
# 分析远程仓库
/spec-analyzer https://github.com/xxx/yyy

# 分析本地仓库
/spec-analyzer /path/to/repo

# 指定选项
/spec-analyzer https://github.com/xxx/yyy --output ./docs --depth deep
```

### 2. optimize-skill

系统化优化 Claude Code Skill 的质量：

- 识别 skill 的问题和改进点
- 生成具体的优化建议
- 验证优化效果

```bash
# 优化指定的 skill
/optimize-skill <skill-name>

# 优化时使用特定测试案例
/optimize-skill spec-analyzer --test-case https://github.com/gsd-build/get-shit-done

# 指定优化焦点
/optimize-skill spec-analyzer --focus output-quality
```

## 安装

```bash
j-skills link spec-analyzer
j-skills link optimize-skill
```

## 项目结构

```
jacky-spec-analyzer/
├── skills/
│   ├── spec-analyzer/
│   │   └── SKILL.md          # 核心分析 skill
│   └── optimize-skill/
│       └── SKILL.md          # Skill 优化 skill
├── workflows/
│   └── optimize-analyzer.md  # 优化工作流（手动）
├── templates/
│   ├── analysis-template.md  # 分析模板
│   └── html/                 # HTML 输出模板
└── examples/
    └── gsd-analysis/         # GSD 分析示例
```

## 输出内容

分析完成后，会生成以下文件：

| 文件 | 说明 |
|------|------|
| `overview.md` | 项目概览 |
| `problems.md` | 问题分析（要解决什么问题、现有痛点） |
| `solutions.md` | 解决方案（核心思路、工作流程） |
| `technical-points.md` | 技术要点（核心概念、关键实现） |
| `html/` | 可交互 HTML（用于 GitHub Pages） |

## License

MIT
