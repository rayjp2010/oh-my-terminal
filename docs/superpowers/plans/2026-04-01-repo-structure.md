# oh-my-terminal Repo Structure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Set up the oh-my-terminal repo with per-tool directories, copy in the user's existing config files, write a README with symlink instructions, and replace the originals with symlinks.

**Architecture:** Flat repo with one directory per tool. Each directory holds the tool's config file(s). A README documents the symlink table and setup commands. No scripts, no tooling — just files and a guide.

**Tech Stack:** Git, shell (zsh), markdown.

---

## File Structure

```
oh-my-terminal/
├── README.md              — repo description, symlink table, setup commands
├── zsh/
│   └── zshrc              — copied from ~/.zshrc
├── tmux/
│   └── tmux.conf          — copied from ~/.tmux.conf
├── vim/
│   └── vimrc              — copied from ~/.vimrc
├── starship/
│   └── starship.toml      — copied from ~/.config/starship.toml
└── ghostty/
    └── config             — copied from ~/.config/ghostty/config
```

---

### Task 1: Create directory structure and copy config files

**Files:**
- Create: `zsh/zshrc`
- Create: `tmux/tmux.conf`
- Create: `vim/vimrc`
- Create: `starship/starship.toml`
- Create: `ghostty/config`

The user's existing config files live at their standard locations. Copy each one into the repo under its tool directory.

- [ ] **Step 1: Create the five tool directories**

```bash
mkdir -p zsh tmux vim starship ghostty
```

Run from: `/Users/rui.a.ding/workspace/oh-my-terminal`

- [ ] **Step 2: Copy each config file into the repo**

```bash
cp ~/.zshrc zsh/zshrc
cp ~/.tmux.conf tmux/tmux.conf
cp ~/.vimrc vim/vimrc
cp ~/.config/starship.toml starship/starship.toml
cp ~/.config/ghostty/config ghostty/config
```

Run from: `/Users/rui.a.ding/workspace/oh-my-terminal`

- [ ] **Step 3: Verify the files were copied**

```bash
ls -la zsh/zshrc tmux/tmux.conf vim/vimrc starship/starship.toml ghostty/config
```

Expected: All five files exist with non-zero sizes (except `vimrc` which is 0 bytes on the system).

- [ ] **Step 4: Commit**

```bash
git add zsh/ tmux/ vim/ starship/ ghostty/
git commit -m "Add terminal config files

Copy existing configs into per-tool directories:
- zsh/zshrc
- tmux/tmux.conf
- vim/vimrc
- starship/starship.toml
- ghostty/config"
```

---

### Task 2: Write the README

**Files:**
- Create: `README.md`

The README is the main user-facing document. It describes the repo, lists the symlink mappings, provides copy-paste setup commands, and explains how to add new tools.

- [ ] **Step 1: Create README.md**

Write the following to `README.md`:

```markdown
# oh-my-terminal

Centralized terminal configuration files. Configs are activated by symlinking from their expected locations to this repo.

## Setup

Clone this repo, then create symlinks. The commands below assume the repo lives at `~/oh-my-terminal`.

### Symlink table

| Repo path                | Symlink target              |
|--------------------------|-----------------------------|
| `zsh/zshrc`              | `~/.zshrc`                  |
| `tmux/tmux.conf`         | `~/.tmux.conf`              |
| `vim/vimrc`              | `~/.vimrc`                  |
| `starship/starship.toml` | `~/.config/starship.toml`   |
| `ghostty/config`         | `~/.config/ghostty/config`  |

### Commands

Back up any existing files first, then create the symlinks:

```sh
# Back up existing configs
mkdir -p ~/.config-backup
[ -f ~/.zshrc ] && mv ~/.zshrc ~/.config-backup/
[ -f ~/.tmux.conf ] && mv ~/.tmux.conf ~/.config-backup/
[ -f ~/.vimrc ] && mv ~/.vimrc ~/.config-backup/
[ -f ~/.config/starship.toml ] && mv ~/.config/starship.toml ~/.config-backup/
[ -f ~/.config/ghostty/config ] && mv ~/.config/ghostty/config ~/.config-backup/

# Create symlinks
ln -s ~/oh-my-terminal/zsh/zshrc ~/.zshrc
ln -s ~/oh-my-terminal/tmux/tmux.conf ~/.tmux.conf
ln -s ~/oh-my-terminal/vim/vimrc ~/.vimrc
mkdir -p ~/.config
ln -s ~/oh-my-terminal/starship/starship.toml ~/.config/starship.toml
mkdir -p ~/.config/ghostty
ln -s ~/oh-my-terminal/ghostty/config ~/.config/ghostty/config
```

## Adding a new tool

1. Create a directory named after the tool (e.g., `alacritty/`).
2. Add the config file inside it.
3. Add a row to the symlink table above.
4. Add the `ln -s` command to the commands section.
```

- [ ] **Step 2: Verify the README renders correctly**

```bash
cat README.md
```

Skim the output to confirm the markdown table and code blocks look correct.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add README with symlink table and setup commands"
```

---

### Task 3: Replace original config files with symlinks

**Files:**
- Modify (on disk, outside repo): `~/.zshrc`, `~/.tmux.conf`, `~/.vimrc`, `~/.config/starship.toml`, `~/.config/ghostty/config`

This task activates the repo by replacing the user's original config files with symlinks pointing into the repo. Backs up originals first.

- [ ] **Step 1: Back up existing config files**

```bash
mkdir -p ~/.config-backup
cp ~/.zshrc ~/.config-backup/zshrc.bak
cp ~/.tmux.conf ~/.config-backup/tmux.conf.bak
cp ~/.vimrc ~/.config-backup/vimrc.bak
cp ~/.config/starship.toml ~/.config-backup/starship.toml.bak
cp ~/.config/ghostty/config ~/.config-backup/ghostty-config.bak
```

Expected: All five `.bak` files created in `~/.config-backup/`.

- [ ] **Step 2: Verify backups exist**

```bash
ls -la ~/.config-backup/
```

Expected: Five `.bak` files present.

- [ ] **Step 3: Remove originals and create symlinks**

```bash
rm ~/.zshrc
ln -s ~/oh-my-terminal/zsh/zshrc ~/.zshrc

rm ~/.tmux.conf
ln -s ~/oh-my-terminal/tmux/tmux.conf ~/.tmux.conf

rm ~/.vimrc
ln -s ~/oh-my-terminal/vim/vimrc ~/.vimrc

rm ~/.config/starship.toml
ln -s ~/oh-my-terminal/starship/starship.toml ~/.config/starship.toml

rm ~/.config/ghostty/config
ln -s ~/oh-my-terminal/ghostty/config ~/.config/ghostty/config
```

- [ ] **Step 4: Verify symlinks are correct**

```bash
ls -la ~/.zshrc ~/.tmux.conf ~/.vimrc ~/.config/starship.toml ~/.config/ghostty/config
```

Expected: Each file shows as a symlink (`l` permission prefix) pointing to the corresponding path under `~/oh-my-terminal/`.

- [ ] **Step 5: Verify configs still work**

```bash
source ~/.zshrc
```

Expected: Shell reloads without errors. (Other tools — tmux, vim, starship, ghostty — can be verified by opening them normally.)
