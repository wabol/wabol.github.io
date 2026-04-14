---
title: Vim search configuration
date: 2025-02-14 08:31:00 -0700
categories: [vim]
tags: 
Pin:
---

This note records the full `~/.vimrc` setup and the basic search usage.

## Full vimrc config

```vim
syntax on " Enable syntax highlighting
set number
set relativenumber
set tabstop=4
set shiftwidth=4
set expandtab
set cursorline

" Search behavior
set ignorecase
set smartcase
set incsearch
set hlsearch
set wrapscan

" Clear search highlight quickly
nnoremap <silent> <Esc><Esc> :nohlsearch<CR>

" Keep next/prev search results centered
nnoremap n nzzzv
nnoremap N Nzzzv

" Use ripgrep for project-wide search when available
if executable('rg')
  set grepprg=rg\ --vimgrep\ --smart-case
  set grepformat=%f:%l:%c:%m

  if !exists(':Rg')
    command! -nargs=+ Rg execute 'silent grep! ' . <q-args> | copen
  endif

  cnoreabbrev <expr> rg getcmdtype() ==# ':' && getcmdline() ==# 'rg' ? 'Rg' : 'rg'
endif
```

The block above is the full current config, so it can be copied to a new machine directly.

## Search config only

```vim
" Search behavior
set ignorecase
set smartcase
set incsearch
set hlsearch
set wrapscan

" Clear search highlight quickly
nnoremap <silent> <Esc><Esc> :nohlsearch<CR>

" Keep next/prev search results centered
nnoremap n nzzzv
nnoremap N Nzzzv

" Use ripgrep for project-wide search when available
if executable('rg')
  set grepprg=rg\ --vimgrep\ --smart-case
  set grepformat=%f:%l:%c:%m

  if !exists(':Rg')
    command! -nargs=+ Rg execute 'silent grep! ' . <q-args> | copen
  endif

  cnoreabbrev <expr> rg getcmdtype() ==# ':' && getcmdline() ==# 'rg' ? 'Rg' : 'rg'
endif
```

## What it does

- `ignorecase`: search ignores case by default
- `smartcase`: if the search text has uppercase letters, Vim uses case-sensitive search
- `incsearch`: show matches while typing
- `hlsearch`: highlight all matches
- `wrapscan`: search continues from the top after the end of the file
- `<Esc><Esc>`: clear search highlight
- `n` and `N`: jump to next or previous result and keep it near the center of the screen

## Search in current file

### Normal search

```vim
/word
```

- `n`: next match
- `N`: previous match

### Search word under cursor

- `*`: search the current word forward, whole word match
- `#`: search the current word backward, whole word match
- `g*`: search the current word forward, partial match
- `g#`: search the current word backward, partial match

Example:

- `count` matches only `count` with `*`
- `count` can also match `counter` or `item_count` with `g*`

## Search in the whole project

Vim is configured to use `rg` as the backend for `:grep`.

### Method 1: built-in grep command

```vim
:grep word
```

This searches from the current directory and puts results into the quickfix list.

### Method 2: custom command

```vim
:Rg word
```

This does the same search and also opens the quickfix window.

### Lowercase shortcut

```vim
:rg word
```

This works because command-line abbreviation changes `:rg` to `:Rg`.

## Quickfix usage

```vim
:copen
:cclose
:cnext
:cprev
```

- `:copen`: open the result list
- `:cclose`: close the result list
- `:cnext`: jump to the next result
- `:cprev`: jump to the previous result

## Notes

- `:grep` is a built-in Vim command, so it can be lowercase
- `:Rg` is a user-defined command, so its real name must start with uppercase
- `:rg` is not a real command name; it is only an input shortcut

## Reload vimrc

After changing `~/.vimrc`, reload it with:

```vim
:source ~/.vimrc
```