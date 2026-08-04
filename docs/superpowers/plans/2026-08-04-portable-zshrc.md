# Portable Compact Zshrc Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the generated, machine-specific zshrc with a compact portable configuration that preserves general shell behavior and aliases.

**Architecture:** One native Zsh file keeps generic history and input preferences, the retained aliases, a portable Zim bootstrap derived from Homebrew’s prefix, and guarded Mise/Starship activation. Zim owns the Zoxide module through `.zimrc`; no hard-coded paths or application-specific shell setup remain.

**Tech Stack:** Zsh, Homebrew, Zim Framework, Mise, Starship.

---

## File Structure

| File | Responsibility |
| --- | --- |
| `zsh/zshrc` | Portable interactive-shell preferences, aliases, Zim bootstrap, and managed-tool initialization. |

`zsh/zimrc` is intentionally out of scope. It already has a separate,
uncommitted synchronization change and must not be staged by this plan.

### Task 1: Establish the machine-specific baseline

**Files:**
- Test: `zsh/zshrc`

- [ ] **Step 1: Run static checks against the current zshrc**

Run:

```bash
rg -n '/Users/|/opt/homebrew|google-cloud-sdk|Browser-Use|PNPM_HOME|nvim' zsh/zshrc
zsh -n zsh/zshrc
```

Expected: the search finds user-name, Homebrew-prefix, Google Cloud SDK,
Browser Use, PNPM, and editor-specific content; syntax checking exits 0.

- [ ] **Step 2: Record the expected failing boundary**

Run: `! rg -n '/Users/|/opt/homebrew|google-cloud-sdk|Browser-Use|PNPM_HOME|nvim' zsh/zshrc`

Expected: exit status 1 because the current configuration is intentionally not
portable yet.

### Task 2: Replace zshrc with the portable compact configuration

**Files:**
- Modify: `zsh/zshrc`

- [ ] **Step 1: Replace the generated configuration with the focused Zsh file**

```zsh
setopt HIST_IGNORE_ALL_DUPS
bindkey -e
WORDCHARS=${WORDCHARS//[\/]}

ZSH_AUTOSUGGEST_MANUAL_REBIND=1
ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets)

alias src='cd $HOME/workspace'
alias gst='git status'
alias gco='git checkout'
alias gcob='git checkout -b'
alias gb='git branch -vv'
alias gaa='git add .'
alias ga='git add'
alias gcm='git commit -m'
alias gp='git pull'
alias gps='git push'
alias gpsf='git push -f'
alias gbd='git branch -D'
alias gss='git stash save'
alias grb='git rebase -i '
alias gcp='git cherry-pick '
alias glog='git log --reverse'

export PATH="$HOME/.local/bin:$PATH"

ZIM_HOME=${ZDOTDIR:-${HOME}}/.zim
ZIM_CONFIG_FILE=${ZIM_CONFIG_FILE:-${ZDOTDIR:-${HOME}}/.zimrc}

if [[ -r ${ZIM_CONFIG_FILE} && ! ${ZIM_HOME}/init.zsh -nt ${ZIM_CONFIG_FILE} ]] && (( ${+commands[brew]} )); then
  zimfw_script="$(brew --prefix zimfw 2>/dev/null)/share/zimfw.zsh"
  [[ -r ${zimfw_script} ]] && source "${zimfw_script}" init
fi

[[ -r ${ZIM_HOME}/init.zsh ]] && source "${ZIM_HOME}/init.zsh"

if (( ${+commands[mise]} )); then
  eval "$(mise activate zsh)"
fi

if (( ${+commands[starship]} )); then
  eval "$(starship init zsh)"
fi
```

Keep the alias values exactly as shown. Do not re-add an editor alias,
application-specific `PATH` entry, cloud-SDK sourcing, PNPM setup, a second
Zoxide initialization, or comments beyond the configuration itself.

- [ ] **Step 2: Validate Zsh syntax and the portability boundary**

Run:

```bash
zsh -n zsh/zshrc
! rg -n '/Users/|/opt/homebrew|google-cloud-sdk|Browser-Use|PNPM_HOME|nvim|zoxide init' zsh/zshrc
rg -n '^alias (src|gst|gco|gcob|gb|gaa|ga|gcm|gp|gps|gpsf|gbd|gss|grb|gcp|glog)=' zsh/zshrc
```

Expected: syntax checking exits 0, no unwanted machine/application-specific
text is present, and all 16 retained aliases are listed once.

- [ ] **Step 3: Commit only the zshrc change**

```bash
git add zsh/zshrc
git commit -m "refactor: simplify portable zshrc"
```

Do not stage `zsh/zimrc`.

### Task 3: Verify the delivered configuration and preserve pending work

**Files:**
- Verify: `zsh/zshrc`
- Preserve: `zsh/zimrc`

- [ ] **Step 1: Run final focused checks**

Run:

```bash
zsh -n zsh/zshrc
! rg -n '/Users/|/opt/homebrew|google-cloud-sdk|Browser-Use|PNPM_HOME|nvim|zoxide init' zsh/zshrc
git diff --check HEAD
git diff --name-only
```

Expected: the zshrc checks succeed. After the zshrc commit, the only remaining
modified tracked file is `zsh/zimrc`; it is the separately requested sync and
is not changed by this plan.

- [ ] **Step 2: Commit any zshrc-only verification correction**

If verification required a correction, commit only that correction:

```bash
git add zsh/zshrc
git commit -m "chore: verify portable zshrc"
```

If no correction was required, do not create an empty commit.
