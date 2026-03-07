# Chezmoi Troubleshooting Guide

## Diagnostic Commands

```bash
# Comprehensive diagnosis
chezmoi doctor

# View status
chezmoi status

# View differences
chezmoi diff

# View template data
chezmoi data

# View source file path
chezmoi source-path <file>

# View target file path
chezmoi target-path <file>
```

## Common Issues

### Issue 1: File Exists

**Symptom**:
```
chezmoi: warning: <target> has changed since chezmoi last wrote it
```

**Solution**:

```bash
# View differences
chezmoi diff ~/.zshrc

# Option A: Sync current file content to source
chezmoi re-add ~/.zshrc

# Option B: Force overwrite target file
chezmoi apply --force ~/.zshrc

# Option C: Backup then apply
cp ~/.zshrc ~/.zshrc.backup
chezmoi apply ~/.zshrc
```

---

### Issue 2: Permission Denied

**Symptom**:
```
chezmoi: open <file>: permission denied
```

**Solution**:

```bash
# Check permissions
ls -la <file>

# Fix permissions
chezmoi apply --verbose

# If sudo needed, execute manually
sudo chezmoi apply /etc/some-config

# Or mark as needing sudo (in source file)
# Add prefix: run_as_root_ (not recommended)
```

---

### Issue 3: Template Error

**Symptom**:
```
chezmoi: template: <file>:<line>: <error>
```

**Solution**:

```bash
# View specific error location
chezmoi cat ~/.zshrc

# Common errors and fixes:

# Error 1: Undefined variable
# Fix: Provide default value
{{ .undefined_var | default "fallback" }}

# Error 2: Bracket mismatch
# Fix: Check {{ and }} pairing

# Error 3: Function doesn't exist
# Fix: Check function name spelling
```

---

### Issue 4: Git Related Issues

**Symptom A**: Cannot pull updates
```
chezmoi: git: exit status 1
```

**Solution**:
```bash
# Enter source directory and handle manually
cd $(chezmoi source-path)
git status

# If conflict
git pull --rebase
# Or
git fetch origin && git reset --hard origin/main
```

**Symptom B**: Cannot push
```
chezmoi: git: exit status 128
```

**Solution**:
```bash
cd $(chezmoi source-path)
git remote -v  # Check remote repo
git push -u origin main  # Set upstream branch
```

---

### Issue 5: Initialization Failed

**Symptom**:
```
chezmoi init: exit status 1
```

**Solution**:

```bash
# Check network
curl -I https://github.com

# Check repo address
chezmoi init --debug <url>

# Use HTTPS instead of SSH
chezmoi init https://github.com/user/dotfiles.git

# Manual clone
git clone <url> ~/.local/share/chezmoi
chezmoi init
```

---

### Issue 6: Config Not Applied

**Symptom**: No changes after applying

**Solution**:

```bash
# Check if applied correctly
chezmoi status

# Check target file
cat ~/.zshrc

# Check source file
cat $(chezmoi source-path ~/.zshrc)

# Force re-apply
chezmoi apply --force ~/.zshrc

# Check for .tmpl suffix (template file)
ls ~/.local/share/chezmoi/ | grep zshrc
```

---

### Issue 7: Sensitive File Permission Error

**Symptom**: SSH key permissions too open
```
Permissions 0644 for '~/.ssh/id_rsa' are too open
```

**Solution**:

```bash
# Re-add using private_ prefix
chezmoi add --private ~/.ssh/id_rsa

# Or manually fix permissions
cd $(chezmoi source-path)
chmod 600 private_dot_ssh/id_rsa
```

---

### Issue 8: Script Not Executing

**Symptom**: run_ script didn't run

**Solution**:

```bash
# Check script permissions
chezmoi apply --verbose

# Check script syntax
bash -n $(chezmoi source-path)/run_script.sh

# Manual execution test
$(chezmoi source-path)/run_script.sh

# Check prefix is correct
# run_once_ only executes first time
# run_onchange_ only executes when content changes
# run_ executes on every apply
```

---

### Issue 9: Ignore File Not Working

**Symptom**: Files in .chezmoiignore still being added

**Solution**:

```bash
# Check .chezmoiignore location
cat $(chezmoi source-path)/.chezmoiignore

# Check syntax (use gitignore syntax)
# Correct: *.log
# Correct: README.md
# Wrong: /README.md (unless in subdirectory)

# Re-add parent directory
chezmoi add ~/.config
```

---

### Issue 10: External Files Not Updating

**Symptom**: Repos referenced with `externals` not updating

**Solution**:

```bash
# Force update external repos
chezmoi apply --refresh-externals

# Or
chezmoi external
```

---

### Issue 11: chezmoi cd Command Hangs

**Symptom**: After executing `chezmoi cd && git status`, command keeps running with no response

**Cause**: `chezmoi cd` is an **interactive command** that starts a new shell and switches to source directory, does not exit automatically. Therefore commands after `&&` never execute.

**Solution**:

```bash
# ❌ Wrong: This starts interactive shell and hangs
chezmoi cd && git status

# ✅ Correct way 1: Use source-path to get path
cd $(chezmoi source-path) && git status

# ✅ Correct way 2: Use path directly
cd ~/.local/share/chezmoi && git status

# ✅ Correct way 3: Use chezmoi git subcommand
chezmoi git status
```

**Summary**:
- `chezmoi cd` - Interactive, for manual directory operations
- `$(chezmoi source-path)` - Non-interactive, for getting path in scripts

---

### Issue 12: chezmoi edit/merge Hangs in Scripts

**Symptom**: Executing `chezmoi edit` or `chezmoi merge` in scripts or CI/CD hangs

**Cause**: These commands start interactive editors (like vim) or merge tools, waiting for user input.

**Solution**:

```bash
# ❌ Wrong: Will hang in scripts
chezmoi edit ~/.zshrc
chezmoi merge file
chezmoi edit-config

# ✅ Correct way 1: Directly edit source file
vim $(chezmoi source-path ~/.zshrc)

# ✅ Correct way 2: Use echo/printf to modify
echo "alias ll='ls -la'" >> $(chezmoi source-path ~/.zshrc)

# ✅ Correct way 3: Force apply (skip merge)
chezmoi apply --force file

# ✅ Correct way 4: Set non-interactive editor (CI environment)
export EDITOR="cat"
chezmoi edit ~/.zshrc  # Now won't hang, but won't actually edit
```

---

## Recovery and Reset

### Complete chezmoi Reset

```bash
# 1. Backup current config
cp -r ~/.local/share/chezmoi ~/chezmoi-backup

# 2. Clear state
chezmoi state delete-bucket --bucket=entryState

# 3. Re-initialize
chezmoi init --apply <url>
```

### Remove chezmoi

```bash
# 1. Stop managing (keep files)
chezmoi unmanaged | xargs -I{} rm ~/.local/share/chezmoi/{}

# 2. Complete removal (dangerous!)
chezmoi purge
```

---

## Getting Help

```bash
# View help
chezmoi --help
chezmoi <command> --help

# View docs
chezmoi docs

# View examples
chezmoi execute-template --help
```

## Submit Issue

If above methods don't resolve:

1. Run `chezmoi doctor` and save output
2. Run failing command with `--debug`
3. Visit https://github.com/twpayne/chezmoi/issues

## Quick Checklist

- [ ] Run `chezmoi doctor` to check environment
- [ ] Use `chezmoi status` to view current status
- [ ] Use `chezmoi diff` to view specific changes
- [ ] Check source file permissions (especially private_ files)
- [ ] Verify template syntax (use `chezmoi cat`)
- [ ] Check git remote repo configuration
- [ ] View detailed logs (use `--verbose` or `--debug`)
- [ ] Check if using interactive commands (`chezmoi cd`, `edit`, `merge`)
