---
title: "Linux tools that you never knew you needed"
date: 2021-04-07T09:45:59+07:00
draft: false
tags: ["linux", "tips", "tech"]
comments: true
toc: true
---

## bat - (cat alternative)

- [bat](https://github.com/sharkdp/bat): A cat(1) clone with syntax highlighting and Git integration.
- Example:

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/bat.png" >}}

## fd - (find alternative)

- [fd](https://github.com/sharkdp/fd): a simple, fast and user-friendly alternative to `find`.
- Examples:

{{< figure class="figure" src="https://raw.githubusercontent.com/sharkdp/fd/master/doc/screencast.svg" >}}

## httpie - (wget/curl alternative)

- [httpie](https://httpie.io): a user-friendly command-line HTTP client for the API era. It comes with JSON support, syntax highlighting, persistent sessions, wget-like downloads, plugins, and more.
- Examples:

```bash
# Hello world
$ http httpie.io/hello
# Custom HTTP method, HTTP headers and JSON data
$ http PUT pie.dev/put X-API-Token:123 name=John
# Submitting forms
$ http -f POST pie.dev/post hello=World
# Upload a file using redirect input
$ http pie.dev/post < files/data.json
# ...
# For more examples, check out: https://httpie.io
```

## ripgrep - (grep alternative)

- [ripgrep](https://github.com/BurntSushi/ripgrep): a faster `grep`. ripgrep is a line-oriented search tool that recursively searches your current directory for a regex pattern. By default, ripgrep will respect your .gitignore and automatically skip hidden files/directories and binary files.
- [Benchmark](https://github.com/BurntSushi/ripgrep#quick-examples-comparing-tools).
- Examples:

```bash
# Basic use
$ rg fast README.md
# Regular expressions
$ rg 'fast\w+' README.md
# Recursive search - recursively searching the directory (current directory is default)
$ rg 'fn write\('
# ...
# For more examples, checkout: https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md
```

## delta

- [delta](https://github.com/dandavison/delta): Code evolves, and we all spend time studying diffs. Delta aims to make this both efficient and enjoyable: it allows you to make extensive changes to the layout and styling of diffs, as well as allowing you to stay arbitrarily close to the default git/diff output.
- Diff/git diff doesn't show you exactly what was changed.

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/git-diff.png" >}}

- Delta shows within-line highlights based on a Levenshtein edit inference algorithm.

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/delta.png" >}}

- By default, delta restructures the git output slightly to make the hunk markers human-readable:

{{< figure class="figure" src="https://user-images.githubusercontent.com/52205/81059276-254cf980-8e9e-11ea-95c3-8b757a4c11b5.png" >}}

- Example config:

```bash
[core]
    pager: delta

[delta]
    plus-style: "syntax #98c379"
    minus-style: "syntax #e06c75"
    syntax-theme: OneHalfDark
    navigate: true
    features: line-numbers decorations

[interactive]
    diffFilter: delta --color-only
```

- Completely replace `diff` with `delta`:

```bash
alias diff="delta"
```

## z

- Tired for `cd`ing into the same directories over and over? Save your time with `z` command!
- [z](https://github.com/rupa/z): jump around. Z is a shell script that makes jumping around your file directory pleasantly simple. Instead of trying to remember the exact path of where you need to go, or worse, `cd`ing into the next directory followed by `ls`ing and then cding again over and over (we’ve all been there), Z allows you to “lazy type” where you want to go and it’ll handle the rest.
- Examples:

```bash
# Takes me to my workspace folder from anywhere.
$ z workspace
```

## fzf

- [fzf](https://github.com/junegunn/fzf): fzf is a general-purpose command-line fuzzy finder.
- It's an interactive Unix filter for command-line that can be used with any list; files, command history, processes, hostnames, bookmarks, git commits, etc.
- Examples:

```bash
# Read the list from STDIN and write the selected item to STDOUT
$ find * -type f | fzf > selected
$ vim $(fzf)
# COMMAND **<TAB>
# Files under the current directory
# - You can select multiple items with TAB key
$ vim **<TAB>

# Files under parent directory
$ vim ../**<TAB>

# Files under parent directory that match `fzf`
$ vim ../fzf**<TAB>

# Files under your home directory
$ vim ~/**<TAB>


# Directories under current directory (single-selection)
$ cd **<TAB>

# Directories under ~/github that match `fzf`
$ cd ~/github/fzf**<TAB>

# Can select multiple processes with <TAB> or <Shift-TAB> keys
$ kill -9 <TAB>

$ unset **<TAB>
$ export **<TAB>
$ unalias **<TAB>
# ...
# For more examples, checkout: https://github.com/junegunn/fzf
```

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/fzf-vim.png" >}}

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/fzf-cd.png" >}}

## thefuck

- [thefuck](https://github.com/nvbn/thefuck): Magnificent app which corrects your previous console command.

{{< figure class="figure" src="https://raw.githubusercontent.com/nvbn/thefuck/master/example.gif" >}}

- Examples:

```bash
$ apt-get install vim
E: Could not open lock file /var/lib/dpkg/lock - open (13: Permission denied)
E: Unable to lock the administration directory (/var/lib/dpkg/), are you root?

$ fuck
sudo apt-get install vim [enter/↑/↓/ctrl+c]
Reading package lists... Done
...
# ...
# For more examples, check out: https://github.com/nvbn/thefuck
```

## exa - (ls alternative)

- [exa](https://the.exa.website): A modern replacement for ls. `exa` is an improved file lister with more features and better defaults. It uses colours to distinguish file types and metadata. It knows about symlinks, extended attributes, and Git. And it’s small, fast, and just one single binary.
- Examples:

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/exa.png" >}}

- Completely replace `ls` with `exa`:

```bash
alias ls="exa"
```

{{< figure class="figure" src="/photos/linux-tools-that-you-never-knew-you-needed/exa-ls.png" >}}
