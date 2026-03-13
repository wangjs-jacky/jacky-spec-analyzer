# jacky-spec-analyzer

分析 spec-driven 仓库，提取核心知识并生成有条理的文档。

## 功能

- 分析项目要解决的问题
- 提取解决方案和工作流程
- 提取关键技术要点
- 输出 Markdown 和可交互 HTML

## 使用方式

```bash
# 分析远程仓库
/spec-analyzer https://github.com/xxx/yyy

# 分析本地仓库
/spec-analyzer /path/to/repo

# 指定选项
/spec-analyzer https://github.com/xxx/yyy --output ./docs --depth deep
```

## 安装

```bash
j-skills link spec-analyzer
```

## 项目结构

```
jacky-spec-analyzer/
├── skills/
│   └── spec-analyzer/
│       └── SKILL.md          # 核心分析 skill
├── workflows/
│   └── optimize-analyzer.md  # 优化工作流
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
