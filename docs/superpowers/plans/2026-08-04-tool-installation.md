# Tool Installation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make this repository a copy-based source of truth for macOS tool configuration and installation manifests.

**Architecture:** Homebrew installs Mise, Homebrew-only command-line tools, and desktop applications from `Brewfile`. Mise installs languages, runtimes, and developer CLIs from `mise/config.toml`; the README directs users to copy that configuration and each terminal config into its active location, without symlinks or an installation script. The Zsh configuration activates Homebrew-installed Mise before initializing Mise-managed Starship.

**Tech Stack:** Homebrew Bundle, Mise, TOML, Zsh, Markdown.

---

## File Structure

| File | Responsibility |
| --- | --- |
| `Brewfile` | Declarative Homebrew-only formula and cask inventory. |
| `mise/config.toml` | Mise tools and global settings. |
| `zsh/zshrc` | Portable activation order for Mise and Starship. |
| `README.md` | Bootstrap instructions, copy destinations, and maintenance workflow. |

No automated test source is added. These files are declarative configuration, so validation is performed with the owning tools and Zsh syntax checking.

### Task 1: Add the Homebrew manifest

**Files:**
- Create: `Brewfile`

- [ ] **Step 1: Create the manifest with packages that Mise cannot own**

```ruby
brew "mise"
brew "mole"
brew "zimfw"

cask "cc-switch"
cask "chatgpt"
cask "claude"
cask "ghostty"
cask "orbstack"
```

Do not add `usage`: Mise owns it. Do not add languages, runtimes, or CLIs that
are represented in `mise/config.toml`.

- [ ] **Step 2: Validate the manifest against the current Homebrew installation**

Run: `brew bundle check --file Brewfile --verbose`

Expected: exit status 0 and no missing formulae or casks. An installed package
that is not listed, such as the pre-migration Homebrew `usage` formula, does
not make this check fail.

- [ ] **Step 3: Commit the manifest**

```bash
git add Brewfile
git commit -m "build: add Homebrew bundle manifest"
```

### Task 2: Add the Mise configuration

**Files:**
- Create: `mise/config.toml`

- [ ] **Step 1: Create the managed tool and settings configuration**

```toml
[tools]
aws-cli = { version = "latest", symlink_bins = "true" }
claude-code = "latest"
codex = "latest"
gcloud = "latest"
gh = "latest"
glab = "latest"
herdr = "latest"
node = "latest"
"npm:@colbymchenry/codegraph" = "latest"
pnpm = "latest"
python = "latest"
starship = "latest"
usage = "latest"
uv = "latest"
zoxide = "latest"

[settings]
trusted_config_paths = [
    "~/workspace",
]
experimental = true
minimum_release_age = "0"
```

The namespaced Codegraph tool preserves the selected installed tool rather than
its historical duplicate versions. `usage` resolves through Mise's registry;
do not use the Homebrew formula after this migration.

- [ ] **Step 2: Verify Mise can load the repository configuration without changing the real global configuration**

```bash
temporary_config_root="$(mktemp -d)"
mkdir -p "$temporary_config_root/mise"
cp mise/config.toml "$temporary_config_root/mise/config.toml"
XDG_CONFIG_HOME="$temporary_config_root" mise config ls
XDG_CONFIG_HOME="$temporary_config_root" mise install --dry-run
rm -rf "$temporary_config_root"
```

Expected: `mise config ls` lists the temporary `mise/config.toml`; the dry run
exits 0 and lists the declared tools without downloading or installing them.

- [ ] **Step 3: Commit the Mise configuration**

```bash
git add mise/config.toml
git commit -m "build: add Mise tool configuration"
```

### Task 3: Make shell activation portable and ordered

**Files:**
- Modify: `zsh/zshrc:128-141`

- [ ] **Step 1: Replace the current Starship and user-specific Mise initialization**

Replace the existing unconditional Starship initialization and the later
hard-coded `/Users/rui.a.ding/.local/bin/mise` command with this single block
before the `# ENV` section:

```zsh
# Initialize Mise before tools it manages, such as Starship.
if command -v mise >/dev/null 2>&1; then
  eval "$(mise activate zsh)"
fi

if command -v starship >/dev/null 2>&1; then
  eval "$(starship init zsh)"
fi
```

This uses the Homebrew-installed `mise` found on `PATH`, makes Starship
available through Mise before invoking it, and leaves the shell usable if the
bootstrap steps have not yet run.

- [ ] **Step 2: Syntax-check the edited Zsh configuration**

Run: `zsh -n zsh/zshrc`

Expected: exit status 0 with no output.

- [ ] **Step 3: Commit the Zsh portability fix**

```bash
git add zsh/zshrc
git commit -m "fix: initialize Mise before Starship"
```

### Task 4: Rewrite the setup documentation for copying rather than linking

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace the README with the copy-based bootstrap guide**

```markdown
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
- AI and development tools: Claude Code, Codex, Herdr, and Codegraph
- Languages and runtimes: Node.js, Python, pnpm, `uv`, and `usage`
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
[ -e ~/.config/mise/config.toml ] || [ -L ~/.config/mise/config.toml ] && \
  mv ~/.config/mise/config.toml ~/.config/mise/config.toml.before-oh-my-terminal
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
```

- [ ] **Step 2: Inspect the rendered Markdown and destination commands**

Run: `sed -n '1,260p' README.md && git diff --check`

Expected: the README contains no `ln -s` commands, the six source/destination
pairs match the configuration table, and `git diff --check` reports no
whitespace errors.

- [ ] **Step 3: Commit the documentation rewrite**

```bash
git add README.md
git commit -m "docs: document copy-based tool setup"
```

### Task 5: Run the complete non-mutating verification suite

**Files:**
- Verify: `Brewfile`
- Verify: `mise/config.toml`
- Verify: `zsh/zshrc`
- Verify: `README.md`

- [ ] **Step 1: Check the Homebrew declaration**

Run: `brew bundle check --file Brewfile --verbose`

Expected: exit status 0; all `Brewfile` entries are installed.

- [ ] **Step 2: Check the Mise declaration without installation**

Run:

```bash
temporary_config_root="$(mktemp -d)"
mkdir -p "$temporary_config_root/mise"
cp mise/config.toml "$temporary_config_root/mise/config.toml"
XDG_CONFIG_HOME="$temporary_config_root" mise install --dry-run
mise_validation_status=$?
rm -rf "$temporary_config_root"
exit "$mise_validation_status"
```

Expected: exit status 0 and no package downloads or installs.

- [ ] **Step 3: Check shell syntax and documentation consistency**

Run:

```bash
zsh -n zsh/zshrc
! rg -n 'ln -s|/Users/rui\.a\.ding/.local/bin/mise' README.md zsh/zshrc
git diff --check HEAD
git status --short
```

Expected: every command succeeds; no old symlink instruction or user-specific
Mise executable remains; `git status --short` is empty after committing the
previous tasks.

- [ ] **Step 4: Commit any verification-only correction**

If a validation step required a correction, commit only that correction:

```bash
git add Brewfile mise/config.toml zsh/zshrc README.md
git commit -m "chore: verify tool setup manifests"
```

If no correction was required, do not create an empty commit.
