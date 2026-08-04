# Balanced No-Plugin Vim Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the empty vimrc with a useful daily-editing configuration that uses only native Vim features.

**Architecture:** A single Vimscript file enables focused editor defaults, persisted undo, safe cursor restoration, and a small set of non-recursive mappings. Native Vim startup assertions run against a temporary home directory so validation cannot alter the user’s active Vim settings or undo history.

**Tech Stack:** Vim 9.1, Vimscript, shell.

---

## File Structure

| File | Responsibility |
| --- | --- |
| `vim/vimrc` | Native Vim editor behavior, persistent state, autocmds, and mappings. |

No plugin manager, lockfile, language server, or additional runtime dependency
is introduced.

### Task 1: Establish a failing Vim behavior check

**Files:**
- Test: `vim/vimrc`

- [ ] **Step 1: Run startup assertions against the current empty vimrc**

```bash
vim_test_home="$(mktemp -d)"
cleanup_vim_test_home() {
  rm -rf "$vim_test_home"
}
trap cleanup_vim_test_home EXIT

HOME="$vim_test_home" vim -Nu "$PWD/vim/vimrc" -n -es \
  -c 'call assert_equal(1, &number, "number must be enabled")' \
  -c 'call assert_equal(1, &relativenumber, "relative numbers must be enabled")' \
  -c 'call assert_equal(1, &ignorecase, "ignorecase must be enabled")' \
  -c 'call assert_equal(1, &smartcase, "smartcase must be enabled")' \
  -c 'call assert_equal(" ", get(g:, "mapleader", ""), "space must be the leader")' \
  -c 'call assert_notequal("", maparg(" w", "n"), "leader-save mapping must exist")' \
  -c 'if len(v:errors) | cquit 1 | endif' \
  -c 'qall!'
```

Expected: exit status 1. The empty vimrc leaves line numbering, smart search,
the leader key, and the leader-save mapping unset.

- [ ] **Step 2: Confirm the failure names missing editor behavior**

Expected: the Vim assertion output identifies at least `number must be enabled`
before any configuration is added. Do not continue if the command succeeds;
that would mean another configuration source affected the test.

### Task 2: Implement the balanced native Vim configuration

**Files:**
- Modify: `vim/vimrc`

- [ ] **Step 1: Replace the empty file with the native Vimscript configuration**

```vim
" Core behavior
if &compatible
  set nocompatible
endif

filetype plugin indent on
syntax enable

" Navigation and search
set number
set relativenumber
set cursorline
set mouse=a
set scrolloff=8
set sidescrolloff=8
set splitbelow
set splitright
set ignorecase
set smartcase
set incsearch
set hlsearch
set wildmenu
set wildmode=longest:full,full
set completeopt=menuone,noselect

" File handling
set hidden
set autoread
set updatetime=300
set timeoutlen=500
set backspace=indent,eol,start

" Persisted state
if has('persistent_undo')
  let s:undo_dir = expand('~/.vim/undo')
  if !isdirectory(s:undo_dir)
    call mkdir(s:undo_dir, 'p')
  endif
  let &undodir = s:undo_dir
  set undofile
endif

if has('clipboard')
  set clipboard=unnamed
endif

augroup restore_cursor_position
  autocmd!
  autocmd BufReadPost *
        \ if line("'\"") > 1 && line("'\"") <= line('$') |
        \   execute 'normal! g`"' |
        \ endif
augroup END

" Mappings
let mapleader = ' '
nnoremap <silent> <leader>w <Cmd>write<CR>
nnoremap <silent> <leader>q <Cmd>quit<CR>
nnoremap <silent> <Esc> <Cmd>nohlsearch<CR>
nnoremap <silent> <C-h> <C-w>h
nnoremap <silent> <C-j> <C-w>j
nnoremap <silent> <C-k> <C-w>k
nnoremap <silent> <C-l> <C-w>l
```

Do not add `expandtab`, `tabstop`, `shiftwidth`, an autoformatter, plugins, or
language-specific settings.

- [ ] **Step 2: Re-run the behavior assertions and verify they now pass**

```bash
vim_test_home="$(mktemp -d)"
cleanup_vim_test_home() {
  rm -rf "$vim_test_home"
}
trap cleanup_vim_test_home EXIT

HOME="$vim_test_home" vim -Nu "$PWD/vim/vimrc" -n -es \
  -c 'call assert_equal(1, &number, "number must be enabled")' \
  -c 'call assert_equal(1, &relativenumber, "relative numbers must be enabled")' \
  -c 'call assert_equal(1, &ignorecase, "ignorecase must be enabled")' \
  -c 'call assert_equal(1, &smartcase, "smartcase must be enabled")' \
  -c 'call assert_equal(1, &incsearch, "incremental search must be enabled")' \
  -c 'call assert_equal(1, &splitbelow, "splits must open below")' \
  -c 'call assert_equal(1, &splitright, "splits must open right")' \
  -c 'call assert_equal(1, &undofile, "persistent undo must be enabled")' \
  -c 'call assert_equal(" ", get(g:, "mapleader", ""), "space must be the leader")' \
  -c 'call assert_notequal("", maparg(" w", "n"), "leader-save mapping must exist")' \
  -c 'call assert_notequal("", maparg("\<Esc>", "n"), "escape mapping must exist")' \
  -c 'if has("clipboard") | call assert_equal("unnamed", &clipboard, "system clipboard must be enabled") | endif' \
  -c 'if len(v:errors) | cquit 1 | endif' \
  -c 'qall!'
```

Expected: exit status 0 with no assertion failures. The temporary home contains
the generated undo directory and is removed by the trap.

- [ ] **Step 3: Perform a direct clean startup check**

Run: `vim -Nu vim/vimrc -n -es -c 'qall!'`

Expected: exit status 0 with no startup errors.

- [ ] **Step 4: Commit the Vim configuration**

```bash
git add vim/vimrc
git commit -m "feat: add native Vim editing defaults"
```

### Task 3: Verify the delivered configuration stays focused

**Files:**
- Verify: `vim/vimrc`

- [ ] **Step 1: Check syntax, startup behavior, and scope boundaries**

Run:

```bash
vim -Nu vim/vimrc -n -es -c 'qall!'
! rg -n 'plug#|packer|lazy%.nvim|vim%-plug|expandtab|tabstop|shiftwidth' vim/vimrc
git diff --check HEAD
git status --short
```

Expected: every command succeeds. Vim starts with the repository config, no
plugin-manager or global-indentation configuration is present, and the working
tree is clean after the Task 2 commit.

- [ ] **Step 2: Commit any verification-only correction**

If verification required a correction, commit only that correction:

```bash
git add vim/vimrc
git commit -m "chore: verify Vim defaults"
```

If no correction was required, do not create an empty commit.
