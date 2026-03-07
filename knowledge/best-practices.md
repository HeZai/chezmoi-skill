# Chezmoi Best Practices

## Initialization Strategy

### New Machine Initialization

```bash
# Method 1: Using HTTPS
chezmoi init --apply https://github.com/username/dotfiles.git

# Method 2: Using SSH (recommended)
chezmoi init --apply git@github.com:username/dotfiles.git

# Method 3: Step by step
chezmoi init git@github.com:username/dotfiles.git
chezmoi diff    # Preview changes first
chezmoi apply   # Apply after confirmation
```

### First-time Setup

```bash
# 1. Initialize
chezmoi init

# 2. Add basic configs
chezmoi add ~/.zshrc
chezmoi add ~/.gitconfig
chezmoi add ~/.vimrc

# 3. Enter source directory
chezmoi cd

# 4. Create repo and push
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:username/dotfiles.git
git push -u origin main
```

## File Organization

### Recommended Directory Structure

```
~/.local/share/chezmoi/
├── dot_zshrc                    # Basic config
├── dot_gitconfig.tmpl           # Template config
├── dot_vimrc
├── private_dot_ssh/
│   ├── config.tmpl
│   └── authorized_keys.tmpl
├── dot_config/
│   ├── git/
│   │   └── ignore
│   └── nvim/
│       └── init.vim
├── run_once_setup.sh.tmpl       # First-time setup script
├── run_onchange_install.sh.tmpl # Execute on change
└── .chezmoiignore               # Ignore file
```

### Ignore File

Create `.chezmoiignore`:

```
README.md
install.sh
*.md
.DS_Store
```

## Sensitive Information Management

### Using Template Variables

```toml
# chezmoi.toml
[data]
    email = "user@example.com"

[data.work]
    email = "user@company.com"
```

### Using Password Managers

```
# 1Password
{{ onepasswordRead "op://Personal/github/token" }}

# Bitwarden
{{ bitwarden "item" ".login.password" }}

# LastPass
{{ lastpass "item" }}
```

### Using Encryption

```bash
# Encrypt sensitive files
chezmoi encrypt private_file

# Configure auto-encryption
chezmoi add --encrypt private_file
```

## Script Management

### Execution Timing

```
run_once_before_*.sh      # Execute before first apply (once only)
run_onchange_*.sh         # Execute when file changes
run_*.sh                  # Execute on every apply
```

### Script Example

```bash
#!/bin/bash
# run_once_before_install-packages.sh.tmpl

{{ if eq .chezmoi.os "darwin" }}
# macOS
brew install git vim zsh
{{ else if eq .chezmoi.os "linux" }}
# Linux
sudo apt-get update
sudo apt-get install -y git vim zsh
{{ end }}
```

## Multi-machine Management

### Distinguish by Machine Type

```
{{ if eq .chezmoi.hostname "work-laptop" }}
# Work machine config
{{ else if eq .chezmoi.hostname "home-desktop" }}
# Home desktop config
{{ else }}
# Default config
{{ end }}
```

### Distinguish by OS

```
{{ if eq .chezmoi.os "darwin" }}
export PATH="/opt/homebrew/bin:$PATH"
{{ else if eq .chezmoi.os "linux" }}
export PATH="$HOME/.local/bin:$PATH"
{{ end }}
```

### Using Conditional Includes

```
# dot_gitconfig.tmpl
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

## Git Workflow

### Auto-commit

```toml
# chezmoi.toml
[git]
    autoCommit = true
    autoPush = false  # Manual control for push
```

### Commit Conventions

```bash
chezmoi cd

# Add new config
git add dot_zshrc
git commit -m "feat: add zsh aliases for git"

# Update template
git add dot_gitconfig.tmpl
git commit -m "refactor: use template for git email"

# Fix issue
git add dot_vimrc
git commit -m "fix: correct vim color scheme"
```

## Common Issues

### File Permissions

```bash
# Ensure sensitive files have correct permissions
chezmoi add --private ~/.ssh/config
# Or manually mark
chmod 600 $(chezmoi source-path)/private_dot_ssh/config
```

### Symbolic Links

```bash
# Create symbolic link (don't copy content)
chezmoi add --template --autotemplate ~/.config/nvim
# Then modify source file to symlink_ prefix
```

### Temporary Modifications

```bash
# Temporarily modify target file (don't overwrite)
chezmoi add --empty ~/.zshrc.local
# This way ~/.zshrc.local won't be overwritten
```

## Debugging Tips

```bash
# View template render result
chezmoi cat ~/.zshrc

# View template data
chezmoi data

# Verbose output
chezmoi apply -v

# Dry run
chezmoi apply --dry-run

# Diagnose environment
chezmoi doctor
```
