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

- Formulae: `cliproxyapi`, `mise`, `mole`, `officecli`, and `zimfw`
- Casks: CC Switch, ChatGPT, Claude, draw.io, Ghostty, Google Drive, Obsidian,
  OrbStack, Slack, and Zed

### Mise

`mise/mise-config.toml` installs:

- Cloud and Git tools: AWS CLI, Google Cloud CLI, `gh`, and `glab`
- AI and development tools: Claude Code, Codex, Herdr, and Codegraph
- Languages and runtimes: Node.js, Python, pnpm, `uv`, and `usage`
- Terminal tools: Starship, Yazi, and Zoxide

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

cp mise/mise-config.toml ~/.config/mise/config.toml
mise install
```

The `.before-oh-my-terminal` file is a one-time backup of an existing Mise
configuration. Review and merge any machine-specific settings you need before
removing it.

### 4. Copy terminal configuration files

Run the following from the repository root. These commands overwrite the active
configuration files with the repository versions.

```sh
mkdir -p ~/.config/ghostty ~/.config/yazi
cp zsh/zshrc ~/.zshrc
cp zsh/zimrc ~/.zimrc
cp vim/vimrc ~/.vimrc
cp starship/starship.toml ~/.config/starship.toml
cp ghostty/config ~/.config/ghostty/config
cp yazi/* ~/.config/yazi/
```

### 5. Install Yazi plugins and flavors

`yazi/package.toml` pins the plugins and the Dracula flavor. Download them into
`~/.config/yazi/plugins` and `~/.config/yazi/flavors`:

```sh
ya pkg install
```

### 6. Restart affected applications

Open a new terminal for the Zsh, Mise, Starship, and Zoxide changes. Restart
Ghostty to load its copied configuration.

## Configuration destinations

| Repository file | Active location |
| --- | --- |
| `mise/mise-config.toml` | `~/.config/mise/config.toml` |
| `zsh/zshrc` | `~/.zshrc` |
| `zsh/zimrc` | `~/.zimrc` |
| `vim/vimrc` | `~/.vimrc` |
| `starship/starship.toml` | `~/.config/starship.toml` |
| `ghostty/config` | `~/.config/ghostty/config` |
| `yazi/*` | `~/.config/yazi/` |

`~/.config/yazi/plugins` and `~/.config/yazi/flavors` are downloaded by
`ya pkg install` and are not kept in this repository.

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

## Yazi

Yazi is the terminal file manager, installed through Mise. Run it with `yazi`.
Its configuration is split across four files:

| File | Contents |
| --- | --- |
| `yazi/yazi.toml` | Pane ratio, hidden files, sorting, preview sizes, and the `git` fetchers that put git status in the linemode |
| `yazi/keymap.toml` | `l` and `Enter` use `smart-enter`, `Ctrl-o` reveals in Finder, `!` opens a shell in the current directory |
| `yazi/theme.toml` | Selects the Dracula flavor and defines the git status signs the flavor does not ship |
| `yazi/init.lua` | Loads the `full-border` and `git` plugins |

### Adding a plugin or flavor

Install it with `ya pkg`, which writes the pinned revision into
`~/.config/yazi/package.toml`:

```sh
ya pkg add yazi-rs/plugins:smart-enter
ya pkg add yazi-rs/flavors:dracula
```

Plugins that need to be started add a line to `init.lua`; plugins bound to a key
add a block to `keymap.toml`. Check the plugin's README for which it needs. Copy
the changed files back into this repository:

```sh
cp ~/.config/yazi/package.toml ~/.config/yazi/init.lua ~/.config/yazi/keymap.toml yazi/
```

Run `ya pkg upgrade` to move the pins forward, then copy `package.toml` back
here as well.
