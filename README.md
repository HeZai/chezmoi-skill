# Chezmoi Agent Skill

**English** | [中文](README.zh.md)

---

An intelligent [chezmoi](https://github.com/twpayne/chezmoi) assistant that makes dotfiles management simple, safe, and reproducible.

## Features

- **Smart Intent Recognition**: Automatically understands your configuration management needs
- **Safe Execution Grading**: Read-only operations execute automatically, modifications require confirmation
- **Complete Workflow Support**: Covers initialization, adding, applying, templating, and diagnostics
- **Context Awareness**: Automatically detects chezmoi environment and configuration status

## Usage

### Explicit Commands

```
/chezmoi              # Enter chezmoi assistant mode
/chezmoi status       # Check configuration status
/chezmoi add ~/.zshrc # Add configuration file
```

### Natural Language

```
"I want to manage my zsh config"
"How to sync configs on this machine"
"Add gitconfig to dotfiles"
"Got an error when applying configs"
```

## Workflow Examples

### Add New Configuration File

```
You: Add my vim config
Assistant: Detected ~/.vimrc
          Execute: chezmoi add ~/.vimrc
          ✓ File added as dot_vimrc
```

### Initialize New Machine

```
You: Setup dotfiles on new machine
Assistant: Workflow: New machine initialization
          1. Please provide repository URL:
          2. Preview changes...
          3. Apply configurations...
```

### Create Template

```
You: Make email a template variable
Assistant: Detected hardcoded email
          Suggest converting to: email = "{{ .email }}"
          Convert? [Yes/No]
```

## Safety Notes

- **Read-only operations** execute automatically: status, diff, dump
- **Modification operations** require confirmation: add, apply, re-add
- **High-risk operations** must be explicitly confirmed: purge, init --apply

## File Structure

```
~/.claude/skills/chezmoi/
├── SKILL.md              # Skill definition
├── README.md             # This document
├── README.zh.md          # Chinese documentation
├── knowledge/            # Knowledge base
│   ├── chezmoi-concepts.md
│   ├── template-functions.md
│   └── best-practices.md
└── examples/             # Examples
    ├── init-workflow.md
    ├── template-guide.md
    └── troubleshooting.md
```

## Dependencies

- chezmoi >= 2.0, < 3.0
- git

## Learn More

- [chezmoi Official Documentation](https://www.chezmoi.io/)
- Reference documents in `knowledge/` directory
- [中文文档](README.zh.md)
