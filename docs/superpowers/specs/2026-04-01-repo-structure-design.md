# oh-my-terminal Repo Structure Design

## Purpose

A centralized git repo for terminal configuration files. The repo is the single source of truth; config files are activated on a machine by creating symlinks from their expected locations to the repo.

## Directory Layout

```
oh-my-terminal/
├── README.md
├── zsh/
│   └── zshrc
├── tmux/
│   └── tmux.conf
├── vim/
│   └── vimrc
├── starship/
│   └── starship.toml
└── ghostty/
    └── config
```

### Conventions

- One folder per tool, named after the tool.
- Config files are stored without the leading dot (e.g., `zshrc` not `.zshrc`). The symlink restores the expected name.
- Each folder contains a single config file for now. Adding related files later (e.g., vim plugin lists) is natural — just add them to the tool's folder.

## Symlink Table

| Repo path              | Symlink target               |
|------------------------|------------------------------|
| `zsh/zshrc`            | `~/.zshrc`                   |
| `tmux/tmux.conf`       | `~/.tmux.conf`               |
| `vim/vimrc`            | `~/.vimrc`                   |
| `starship/starship.toml` | `~/.config/starship.toml`  |
| `ghostty/config`       | `~/.config/ghostty/config`   |

## Symlink Strategy

Symlinks point from the expected config location to the repo file. This means:

- Edits happen in the repo (or equivalently, at the symlinked path — same file).
- `git diff` always reflects the current state.
- No risk of repo and active config drifting out of sync.

## README

The README includes:

1. A one-line description of the repo.
2. The symlink table above.
3. Ready-to-copy `ln -s` commands for setup on a new machine.
4. A short note on how to add a new tool (create a folder, add to the table).

## Scope Exclusions

- No install script — symlinks are created manually using commands from the README.
- No per-machine overrides or conditional logic.
- No dependency on external tools (Stow, Chezmoi, etc.).
- No Oh My Zsh directory — the `.zshrc` file is sufficient.
- No `.zprofile` — not needed.
