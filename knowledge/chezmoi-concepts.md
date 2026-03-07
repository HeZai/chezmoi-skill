# Chezmoi Core Concepts

## What is Chezmoi

Chezmoi is a cross-platform dotfiles management tool that helps you sync configuration files across multiple machines.

## Core Concepts

### Source State

The repository directory where configurations are stored, default location: `~/.local/share/chezmoi/`

```
~/.local/share/chezmoi/
├── dot_zshrc           # Source file for ~/.zshrc
├── dot_gitconfig       # Source file for ~/.gitconfig
├── private_dot_ssh/    # ~/.ssh/ directory (private_ prefix indicates sensitive)
│   └── config
└── run_once_setup.sh   # First-run script
```

### Target State

The actual location where configurations are applied, i.e., your home directory: `~/`

### Attributes (Prefix Attributes)

| Prefix | Meaning | Example |
|-----|------|------|
| `dot_` | Maps to `.` | `dot_zshrc` → `.zshrc` |
| `private_` | Sets 0600 permissions | `private_dot_ssh` |
| `executable_` | Sets executable permission | `executable_bin_myscript` |
| `symlink_` | Creates symbolic link | `symlink_dot_vimrc` |
| `run_` | Execute script | `run_setup.sh` |
| `run_once_` | Execute only once | `run_once_install.sh` |
| `create_` | Create empty file | `create_.hushlogin` |
| `remove_` | Remove target file | `remove_.bashrc` |
| `modify_` | Modify existing file | `modify_.vimrc` |

### Templating

Use Go template syntax for configuration differentiation:

```
# dot_gitconfig.tmpl
[user]
    name = {{ .name }}
    email = {{ .email }}
```

Common template functions:
- `{{ .chezmoi.os }}` - Operating system (darwin, linux, windows)
- `{{ .chezmoi.arch }}` - Architecture (amd64, arm64)
- `{{ .chezmoi.hostname }}` - Hostname

## Basic Commands

```bash
# Initialize
chezmoi init                    # Initialize empty repository
chezmoi init --apply $URL       # Initialize from remote repo and apply

# Add files
chezmoi add ~/.zshrc            # Add file
chezmoi add --template ~/.gitconfig  # Add as template

# Edit files
chezmoi edit ~/.zshrc           # Edit source file
chezmoi edit --apply ~/.zshrc   # Edit and apply

# View status
chezmoi status                  # View change status
chezmoi diff                    # View detailed differences

# Apply config
chezmoi apply                   # Apply all changes
chezmoi apply ~/.zshrc          # Apply single file
chezmoi apply -v                # Verbose output

# Update
chezmoi update                  # Pull and apply
chezmoi git pull -- --rebase   # Pull only

# Other
chezmoi cd                      # Enter source directory (⚠️ interactive, will block)
chezmoi doctor                  # Diagnose environment
chezmoi data                    # View template data
chezmoi source-path             # Get source directory path (use in scripts)
```

### ⚠️ Interactive Command Notes

The following commands start interactive sessions and **will block** in scripts or automation:

| Command | Behavior | Script Alternative |
|-----|------|------------|
| `chezmoi cd` | Starts interactive shell | `cd $(chezmoi source-path)` |
| `chezmoi edit <file>` | Opens $EDITOR | `vim $(chezmoi source-path <file>)` |
| `chezmoi edit-config` | Edit config file | Directly edit `~/.config/chezmoi/chezmoi.toml` |
| `chezmoi merge <file>` | Three-way merge | `chezmoi apply --force <file>` |
| `chezmoi ssh <host>` | SSH and initialize | SSH first, then manually run chezmoi init |

#### Correct Usage of chezmoi cd

`chezmoi cd` is an **interactive command** that starts a new shell and switches to the source directory, **does not exit automatically**.

```bash
# ❌ Wrong: This will hang because chezmoi cd starts an interactive shell
chezmoi cd && git status

# ✅ Correct way 1: Use source-path to get path
cd $(chezmoi source-path) && git status

# ✅ Correct way 2: Use path directly
cd ~/.local/share/chezmoi && git status

# ✅ Correct way 3: Use chezmoi git subcommand
chezmoi git status
```

## Configuration

Configuration file location: `~/.config/chezmoi/chezmoi.toml`

```toml
[git]
    autoCommit = true
    autoPush = true

[data]
    name = "Your Name"
    email = "you@example.com"
```
