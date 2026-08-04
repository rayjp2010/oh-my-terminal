# oh-my-terminal

Personal macOS configuration and installation manifests for terminal tools.
The repository is the source of truth: copy its configuration files into their
active locations after setup. It deliberately does not use symlinks or an
installation script.

## Package ownership

Use Homebrew to install Mise, macOS applications, and tools that Mise does not
provide. Use Mise for languages, runtimes, and developer CLIs.

### Homebrew

`Brewfile` installs:

- Formulae: `mise`, `mole`, and `zimfw`
- Casks: CC Switch, ChatGPT, Claude, Ghostty, and OrbStack

### Mise

`mise/config.toml` installs:

- Cloud and Git tools: AWS CLI, Google Cloud CLI, `gh`, and `glab`
- AI and development tools: Claude Code, Codex, Herdr, Codegraph, and OpenSpec
- Languages and runtimes: Node.js, Python, pnpm, Terraform, TFLint, `uv`, and `usage`
- Terminal tools: Starship and Zoxide

## Bootstrap a new Mac

### 1. Install Homebrew

Install Homebrew from the [official installation instructions](https://brew.sh),
then open a new terminal so `brew` is on `PATH`.

### 2. Clone this repository and install Homebrew-managed tools

```sh
git clone https://github.com/rayjp2010/oh-my-terminal.git ~/oh-my-terminal
cd ~/oh-my-terminal
brew bundle --file Brewfile
```

### 3. Copy the Mise configuration and install its tools

```sh
cd ~/oh-my-terminal
mkdir -p ~/.config/mise

if [ -e ~/.config/mise/config.toml ] || [ -L ~/.config/mise/config.toml ]; then
  mv ~/.config/mise/config.toml ~/.config/mise/config.toml.before-oh-my-terminal
fi

cp mise/config.toml ~/.config/mise/config.toml
mise install
```

The `.before-oh-my-terminal` file is a one-time backup of an existing Mise
configuration. Review and merge any machine-specific settings you need before
removing it.

### 4. Copy terminal configuration files

Run the following from the repository root. It moves every existing destination
into a timestamped backup directory, including an old symlink, before copying
the repository version into place.

```sh
backup_dir="$HOME/.config-backup/oh-my-terminal-$(date +%Y%m%d-%H%M%S)"

copy_config() {
  source_path="$1"
  destination_path="$2"
  backup_path="$3"

  if [ -e "$destination_path" ] || [ -L "$destination_path" ]; then
    mkdir -p "$(dirname "$backup_dir/$backup_path")"
    mv "$destination_path" "$backup_dir/$backup_path"
  fi

  mkdir -p "$(dirname "$destination_path")"
  cp "$source_path" "$destination_path"
}

copy_config zsh/zshrc "$HOME/.zshrc" zshrc
copy_config zsh/zimrc "$HOME/.zimrc" zimrc
copy_config tmux/tmux.conf "$HOME/.tmux.conf" tmux.conf
copy_config vim/vimrc "$HOME/.vimrc" vimrc
copy_config starship/starship.toml "$HOME/.config/starship.toml" starship.toml
copy_config ghostty/config "$HOME/.config/ghostty/config" ghostty/config
```

### 5. Restart affected applications

Open a new terminal for the Zsh, Mise, Starship, and Zoxide changes. Restart
Ghostty to load its copied configuration.

## Configuration destinations

| Repository file | Active location |
| --- | --- |
| `mise/config.toml` | `~/.config/mise/config.toml` |
| `zsh/zshrc` | `~/.zshrc` |
| `zsh/zimrc` | `~/.zimrc` |
| `tmux/tmux.conf` | `~/.tmux.conf` |
| `vim/vimrc` | `~/.vimrc` |
| `starship/starship.toml` | `~/.config/starship.toml` |
| `ghostty/config` | `~/.config/ghostty/config` |

## Keeping configurations in sync

Edit the repository file first, copy it to the matching active location, and
commit the change. If you edit an active configuration directly, copy it back
to this repository before committing so the repository remains the source of
truth.

## Zim modules

Modules are managed through the copied `~/.zimrc`. Add a `zmodule` line, then
run `zimfw install`. For example:

```sh
zmodule zsh-users/zsh-autosuggestions
zmodule zsh-users/zsh-syntax-highlighting
zimfw install
```
