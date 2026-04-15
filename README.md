# oh-my-terminal

Centralized terminal configuration files. Configs are activated by symlinking from their expected locations to this repo.

## Setup

Clone this repo, then create symlinks. The commands below assume the repo lives at `~/oh-my-terminal`.

### Symlink table

| Repo path                | Symlink target              |
|--------------------------|-----------------------------|
| `zsh/zshrc`              | `~/.zshrc`                  |
| `zsh/zimrc`              | `~/.zimrc`                  |
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
[ -f ~/.zimrc ] && mv ~/.zimrc ~/.config-backup/
[ -f ~/.tmux.conf ] && mv ~/.tmux.conf ~/.config-backup/
[ -f ~/.vimrc ] && mv ~/.vimrc ~/.config-backup/
[ -f ~/.config/starship.toml ] && mv ~/.config/starship.toml ~/.config-backup/
[ -f ~/.config/ghostty/config ] && mv ~/.config/ghostty/config ~/.config-backup/

# Create symlinks
ln -s ~/oh-my-terminal/zsh/zshrc ~/.zshrc
ln -s ~/oh-my-terminal/zsh/zimrc ~/.zimrc
ln -s ~/oh-my-terminal/tmux/tmux.conf ~/.tmux.conf
ln -s ~/oh-my-terminal/vim/vimrc ~/.vimrc
mkdir -p ~/.config
ln -s ~/oh-my-terminal/starship/starship.toml ~/.config/starship.toml
mkdir -p ~/.config/ghostty
ln -s ~/oh-my-terminal/ghostty/config ~/.config/ghostty/config
```

## Installing the tools

| Tool | Website |
|------|---------|
| Ghostty | https://ghostty.org |
| zsh | https://www.zsh.org |
| Zim (zimfw) | https://zimfw.sh |
| Starship | https://starship.rs |
| tmux | https://github.com/tmux/tmux |
| vim | https://www.vim.org |

### Zim modules

Modules are managed via `~/.zimrc`. Add a module with `zmodule`, then run `zimfw install`. For example, add these lines to `~/.zimrc`:

```sh
zmodule zsh-users/zsh-autosuggestions
zmodule zsh-users/zsh-syntax-highlighting
```

Then install:

```sh
zimfw install
```

## Adding a new tool

1. Create a directory named after the tool (e.g., `alacritty/`).
2. Add the config file inside it.
3. Add a row to the symlink table above.
4. Add the `ln -s` command to the commands section.
