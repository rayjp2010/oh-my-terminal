# Portable Compact Zsh Configuration Design

## Goal

Replace the generated, machine-specific `zsh/zshrc` with a concise Zsh
configuration that contains only portable shell behavior, the retained aliases,
and guarded initialization for the repository’s managed tools.

## Scope

The configuration keeps all existing aliases except `vim='nvim'`, which forces
a specific editor. It retains generic history, keymap, word-navigation,
autosuggestion, and syntax-highlighting preferences. It removes generated
comment blocks and all application-specific or username-specific configuration.

Removed content includes Google Cloud SDK paths and sourcing, Browser Use,
pipx paths, `PNPM_HOME`, the hard-coded `/Users/rui.a.ding` paths, and the
hard-coded `/opt/homebrew` Zim path.

## Structure

`zsh/zshrc` remains a single Zsh file organized into four short sections:

1. General editing and history defaults.
2. Retained aliases.
3. A portable local-bin `PATH` entry and Zim bootstrap.
4. Guarded Mise and Starship initialization.

Comments name those sections and explain the Zim bootstrap. Generated comments,
commented-out options, and tool-specific path comments are removed.

## Zim Bootstrap

The configuration derives `ZIM_HOME` and the active Zim config from `HOME` and
`ZDOTDIR`. If `init.zsh` is missing or older than `.zimrc`, it first checks that
Homebrew is available, then derives the Zim framework script with:

```zsh
$(brew --prefix zimfw)/share/zimfw.zsh
```

The script is sourced only when readable. `init.zsh` is sourced only when it
exists. This keeps the shell usable before Homebrew or Zim installation, while
the documented `brew bundle` setup provides the normal Zim dependency.

The `.zimrc` owns Zim modules, including `zim-zoxide`; zshrc does not run a
second `zoxide init zsh` command.

## Tool Initialization Order

Mise is activated first, but only if its command is on `PATH`. Starship is
initialized second under the same guard, so a Starship installed through Mise
is available. Neither missing tool causes shell startup to fail.

## Non-Goals

- No plugin manager other than the existing Zim setup.
- No automatic tool installation.
- No machine-specific paths, user names, cloud-SDK setup, browser automation,
  Python-environment setup, or PNPM setup.
- No change to `zsh/zimrc` beyond the already separate synchronization work.

## Verification

1. Run `zsh -n zsh/zshrc` for syntax validation.
2. Verify the file contains no `/Users/`, `/opt/homebrew`, Google Cloud SDK,
   Browser Use, `PNPM_HOME`, or `nvim` references.
3. Inspect the aliases and initialization order, then run `git diff --check`.

No active `~/.zshrc` is overwritten during repository verification.
