# Balanced No-Plugin Vim Design

## Goal

Turn the empty `vim/vimrc` into a useful daily editing setup without plugins,
language servers, or an imposed project-wide indentation style.

## Scope

The configuration improves editing ergonomics while retaining normal Vim
workflows and filetype-provided indentation. It does not install or configure
plugin managers, automatically reformat files, add language-specific tooling,
or replace existing Vim commands with a custom editing model.

## Editor Experience

The Vim configuration enables:

- Hybrid line numbers: the current line uses its absolute number and surrounding
  lines use relative numbers.
- Cursor-line highlighting, mouse support, and an eight-line/column context
  margin while moving through a file.
- New splits opening below or to the right of the current window.
- Case-insensitive search that becomes case-sensitive when the query contains
  an uppercase character, with incremental matches and visible match results.
- Wildmenu command-line completion and a useful completion-menu default.
- Hidden buffers, automatic detection of externally changed files, responsive
  updates, and conventional backspace behavior in insert mode.
- Syntax highlighting and Vim's built-in filetype plugin and indentation rules.

No global `expandtab`, `tabstop`, `shiftwidth`, or formatter setting is added.
Each filetype or project keeps control of its indentation policy.

## Persisted State

Vim creates `~/.vim/undo` if needed and enables persistent undo when Vim
supports it. A `BufReadPost` autocmd returns to the last recorded cursor
position when that position remains inside the file.

When Vim has clipboard support, unnamed registers use the system clipboard.
This applies to the installed macOS Vim, which reports `+clipboard`.

## Key Mappings

Space is the leader key. The configuration adds only these normal-mode
mappings:

| Key | Action |
| --- | --- |
| `<leader>w` | Save the current file. |
| `<leader>q` | Quit the current window. |
| `<Esc>` | Clear search highlighting. |
| `Ctrl-h`, `Ctrl-j`, `Ctrl-k`, `Ctrl-l` | Move to the left, below, above, or right split. |

The mappings use non-recursive forms. Existing commands, registers, motions,
and other normal-mode keys remain unchanged.

## Configuration Structure

`vim/vimrc` remains a single native Vimscript file organized into focused
sections: core behavior, navigation and search, file handling, persistent
state, and mappings. An augroup isolates the cursor-restoration autocmd so
reloading the vimrc does not duplicate it.

## Verification

Run Vim with only the repository vimrc and a temporary `HOME`. Assertions check
that the file loads, hybrid line numbering, search defaults, system clipboard
integration when available, persistent undo, and the leader mapping are set as
designed. The temporary home is removed after the command; the user’s Vim
configuration and undo history are not changed.

Also run `vim -Nu vim/vimrc -n -es -c 'qall!'` as a direct startup check and
use `git diff --check` to catch whitespace errors.
