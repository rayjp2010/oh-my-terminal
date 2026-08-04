# Tool Installation and Copy-Based Configuration Design

## Goal

Expand `oh-my-terminal` from terminal-only dotfiles into the source of truth for
tool configuration and a repeatable macOS setup guide. The repository must keep
configuration files as ordinary tracked files; setup copies them into their live
locations and never creates symlinks.

## Chosen Approach

Track a Mise configuration file and a Homebrew `Brewfile`, then document a
manual, copy-based bootstrap sequence in the README.

- Homebrew installs itself separately, then installs Mise, tools unavailable
  through Mise, and macOS applications through `brew bundle`.
- Mise installs languages, runtimes, and command-line tools through
  `mise install` after its repository configuration is copied into place.
- Existing terminal configuration files are copied into their target locations
  after first backing up any destination that is not already the same file.
- No setup script, symlink, credential, shell-history, or application-login
  state is added to the repository.

## Repository Layout

```
oh-my-terminal/
├── Brewfile
├── mise/
│   └── config.toml
├── ghostty/
│   └── config
├── starship/
│   └── starship.toml
├── vim/
│   └── vimrc
├── zsh/
│   ├── zimrc
│   └── zshrc
└── README.md
```

The `mise/` directory follows the same convention as the existing tool
directories: it contains the file without placing it directly in a user home
directory. During setup, `mise/config.toml` is copied to
`~/.config/mise/config.toml`.

## Package Ownership

### Mise

`mise/config.toml` declares the selected tools to manage through Mise, using
`latest` versions. It preserves the existing configured tools and adds selected
installed tools that were not yet declared in the user configuration:

- `aws-cli`, `claude-code`, `codex`, `gcloud`, `gh`, `glab`, and `herdr`
- Node.js, Python, `pnpm`, and `uv`
- Starship and Zoxide
- `npm:@colbymchenry/codegraph`
- `usage`, moved from Homebrew because Mise provides it through its registry

Old duplicate runtime versions are not encoded. The managed configuration
requests one current version per tool and `mise install` resolves it.

The existing Mise settings remain: workspace trust is limited to `~/workspace`,
experimental features are enabled, and release-age filtering is disabled.

### Homebrew

`Brewfile` owns only Homebrew itself, packages not available through Mise, and
macOS desktop applications:

- Formulae: `mise`, `mole`, and `zimfw`
- Casks: `cc-switch`, `chatgpt`, `claude`, `ghostty`, and `orbstack`

`usage` is intentionally absent from `Brewfile` because Mise owns it. `mole`
and `zimfw` remain in Homebrew because they are not present in the Mise
registry.

## Setup Flow

The README describes these steps in this order:

1. Install Homebrew using its official installer if `brew` is unavailable.
2. Clone this repository and run `brew bundle --file Brewfile` from its root.
3. Back up then copy `mise/config.toml` to `~/.config/mise/config.toml`.
4. Run `mise install` to install the declared languages, runtimes, and CLIs.
5. Back up then copy each terminal configuration file to its normal destination
   (`~/.zshrc`, `~/.zimrc`, `~/.vimrc`,
   `~/.config/starship.toml`, and `~/.config/ghostty/config`).
6. Restart the terminal or reload the relevant application.

The commands create parent directories where needed, store backups under a
timestamped directory, and do not delete user data. They use `cp` rather than
`ln -s`; changes made after setup must be copied back into the repository before
being committed.

## README Content

The revised README contains:

- A concise statement that the repository contains configuration and setup
  manifests for macOS terminal tools.
- The Homebrew/Mise ownership rule and the complete package lists.
- Copy-ready bootstrap commands.
- A configuration destination table with source and destination paths.
- A short update workflow: edit in the repository, copy the changed file into
  its active location, and commit the change; copy local changes back before
  committing if they were made outside the repository.
- Existing Zim module guidance, updated to refer to the copied `~/.zimrc`.

## Verification

Verification is documentation and manifest focused:

1. Parse `mise/config.toml` with `mise` and verify `mise install --dry-run`
   accepts every declaration without changing the machine.
2. Run `brew bundle check --file Brewfile` to validate the Homebrew manifest
   against the installed package inventory.
3. Use `git diff --check` and inspect the README commands and destination table
   for path consistency.

No package installation or configuration-file overwrite is performed during
repository verification.
