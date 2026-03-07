# Chezmoi Initialization Workflow Examples

## Scenario 1: Creating Dotfiles from Scratch

### Step 1: Initialize chezmoi

```bash
# Create new chezmoi repository
chezmoi init
```

### Step 2: Add Config Files

```bash
# Add common config files
chezmoi add ~/.zshrc
chezmoi add ~/.gitconfig
chezmoi add ~/.vimrc

# Add sensitive config (auto-sets permissions)
chezmoi add --private ~/.ssh/config
```

### Step 3: Configure Template Variables

Edit `~/.config/chezmoi/chezmoi.toml`:

```toml
[data]
    name = "Your Name"
    email = "you@example.com"
    editor = "vim"
```

### Step 4: Convert Static Config to Template

```bash
# Convert gitconfig to template
chezmoi edit ~/.gitconfig
```

Modify to:

```
# dot_gitconfig.tmpl
[user]
    name = {{ .name }}
    email = {{ .email }}
[core]
    editor = {{ .editor }}
```

### Step 5: Commit to Git

```bash
# Enter source directory
chezmoi cd

# Initialize git repo
git init
git add .
git commit -m "Initial dotfiles commit"

# Add remote repo
git remote add origin git@github.com:username/dotfiles.git
git push -u origin main
```

---

## Scenario 2: Apply Config on New Machine

### Method 1: One-line Initialization

```bash
chezmoi init --apply git@github.com:username/dotfiles.git
```

### Method 2: Step by Step (Recommended)

```bash
# 1. Initialize repo
chezmoi init git@github.com:username/dotfiles.git

# 2. View changes to be applied
chezmoi diff

# 3. Apply config
chezmoi apply

# 4. Verify
chezmoi status
```

### Method 3: Preview First

```bash
# Initialize but don't apply
chezmoi init git@github.com:username/dotfiles.git

# View specific file content
chezmoi cat ~/.zshrc

# Apply only specific file
chezmoi apply ~/.zshrc
```

---

## Scenario 3: Work Machine Special Config

### Set Machine-specific Data

In `~/.config/chezmoi/chezmoi.toml`:

```toml
[data]
    name = "Your Name"
    email = "you@example.com"

[data.work]
    email = "you@company.com"
    vpn_endpoint = "vpn.company.com"
```

### Create Conditional Template

`dot_gitconfig.tmpl`:

```
[user]
    name = {{ .name }}
{{ if eq .chezmoi.hostname "work-laptop" }}
    email = {{ .work.email }}
{{ else }}
    email = {{ .email }}
{{ end }}
```

### Work-specific SSH Config

`private_dot_ssh/config.tmpl`:

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_personal

{{ if eq .chezmoi.hostname "work-laptop" }}
Host work-github
    HostName github.company.com
    User git
    IdentityFile ~/.ssh/id_work

Host *.company.com
    User yourname
    IdentityFile ~/.ssh/id_work
{{ end }}
```

---

## Scenario 4: Auto-install Dependencies

### Create Install Script

`run_once_before_install-packages.sh.tmpl`:

```bash
#!/bin/bash

{{ if eq .chezmoi.os "darwin" }}
# macOS - Use Homebrew
if ! command -v brew &> /dev/null; then
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
fi

brew bundle --file=- <<EOF
brew "git"
brew "vim"
brew "zsh"
brew "chezmoi"
EOF

{{ else if eq .chezmoi.os "linux" }}
# Linux - Use apt
sudo apt-get update
sudo apt-get install -y \
    git \
    vim \
    zsh \
    curl \
    wget

{{ end }}
```

### Set Default Shell

`run_once_after_set-shell.sh`:

```bash
#!/bin/bash

if [[ "$SHELL" != *"zsh"* ]]; then
    chsh -s $(which zsh)
    echo "Default shell changed to zsh, please re-login"
fi
```

---

## Scenario 5: Sync Updates

### Daily Update Workflow

```bash
# 1. Pull latest config
chezmoi update

# 2. Or step by step
chezmoi git pull -- --rebase
chezmoi apply

# 3. View update content
chezmoi diff
```

### Add New Config and Push

```bash
# 1. Add new file
chezmoi add ~/.config/alacritty/alacritty.yml

# 2. Edit (if needed)
chezmoi edit ~/.config/alacritty/alacritty.yml

# 3. Commit
chezmoi cd
git add .
git commit -m "feat: add alacritty config"
git push
```

### Auto-commit Config

Enable in `chezmoi.toml`:

```toml
[git]
    autoCommit = true
    autoPush = false  # Manual control for push timing
```

---

## Complete Example Repository Structure

```
dotfiles/
├── README.md
├── dot_zshrc
├── dot_zshrc.d/
│   ├── aliases.zsh
│   ├── functions.zsh
│   └── path.zsh
├── dot_gitconfig.tmpl
├── dot_vimrc
├── private_dot_ssh/
│   ├── config.tmpl
│   └── authorized_keys.tmpl
├── dot_config/
│   ├── git/
│   │   └── ignore
│   └── nvim/
│       └── init.vim
├── run_once_before_install-packages.sh.tmpl
├── run_once_after_set-shell.sh
└── .chezmoiignore
```

## Quick Reference

```bash
# Initialize
chezmoi init
chezmoi init --apply <url>

# Add
chezmoi add <file>
chezmoi add --template <file>
chezmoi add --private <file>

# Edit
chezmoi edit <file>
chezmoi edit --apply <file>

# Apply
chezmoi apply
chezmoi apply <file>
chezmoi apply -v  # Verbose

# Update
chezmoi update
chezmoi git pull

# Other
chezmoi status
chezmoi diff
chezmoi doctor
chezmoi cd
```
