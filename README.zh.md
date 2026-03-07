# Chezmoi Agent Skill

[English](README.md) | **中文**

---

智能 [chezmoi](https://github.com/twpayne/chezmoi) 助手，让 dotfiles 管理变得简单、安全、可复现。

## 功能特性

- **智能意图识别**：自动理解用户的配置管理需求
- **安全执行分级**：只读操作自动执行，修改操作需确认
- **完整工作流支持**：初始化、添加、应用、模板、诊断全覆盖
- **上下文感知**：自动检测 chezmoi 环境和配置状态

## 使用方式

### 显式命令

```
/chezmoi              # 进入 chezmoi 助手模式
/chezmoi status       # 查看配置状态
/chezmoi add ~/.zshrc # 添加配置文件
```

### 自然语言

```
"我想管理我的 zsh 配置"
"如何同步这台机器的配置"
"把 gitconfig 加入 dotfiles"
"应用配置时报错了"
```

## 工作流示例

### 添加新配置文件

```
你：添加我的 vim 配置
助手：检测到 ~/.vimrc
      执行：chezmoi add ~/.vimrc
      ✓ 文件已添加为 dot_vimrc
```

### 初始化新机器

```
你：在新机器上设置 dotfiles
助手：工作流：新机器初始化
      1. 请提供仓库地址：
      2. 预览变更...
      3. 应用配置...
```

### 创建模板

```
你：把邮箱改成模板变量
助手：检测到硬编码邮箱
      建议转换为：email = "{{ .email }}"
      是否转换？[是/否]
```

## 安全说明

- **只读操作**自动执行：status、diff、dump
- **修改操作**需确认：add、apply、re-add
- **高风险操作**必须显式确认：purge、init --apply

## 文件结构

```
~/.claude/skills/chezmoi/
├── SKILL.md              # 技能定义
├── README.md             # 英文文档
├── README.zh.md          # 本文档
├── knowledge/            # 知识库
│   ├── chezmoi-concepts.md
│   ├── template-functions.md
│   └── best-practices.md
└── examples/             # 示例
    ├── init-workflow.md
    ├── template-guide.md
    └── troubleshooting.md
```

## 依赖

- chezmoi >= 2.0, < 3.0
- git

## 了解更多

- [chezmoi 官方文档](https://www.chezmoi.io/)
- `knowledge/` 目录下的参考文档
- [English Documentation](README.md)
