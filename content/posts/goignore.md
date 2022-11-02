+++
title = "Goignore"
date = 2021-11-26T17:26:44+07:00
tags = ["tech", "golang"]
comments = true
draft = false
+++

{{< quote info >}}
[Goignore](https://github.com/ntk148v/goignore) - A `.gitignore` wizard which gnerates `.gitignore` files from the command line for you. Inspired by [joe](https://github.com/karan/joe)
{{< /quote >}}

## 1. Features

- No installation necessary - just use the binary.
- Works on Linux, Windows & MacOS.
- Interactive user interface with [bubbletea](https://github.com/charmbracelet/bubbletea): Pagination, Filtering, Help...
- Supports all Github-supported [.gitignore files](https://github.com/github/gitignore.git).

## 2. Install

- Download the latest binary from the [Release page](https://github.com/ntk148v/goignore/releases). It's the easiest way to get started with `goignore`.
- Make sure to add the location of the binary to your `$PATH`.

## 3. Usage

- Just run.

```bash
chmod a+x goignore
goignore
```

- At the first time, `goignore` will download the Gitignore templates from Github. It may take a few seconds (depend on your network).

- The list of gitignore templates.

![](https://raw.githubusercontent.com/ntk148v/goignore/master/screenshots/start.png)

- Show help.

![](https://raw.githubusercontent.com/ntk148v/goignore/master/screenshots/help.png)

- Filter.

![](https://raw.githubusercontent.com/ntk148v/goignore/master/screenshots/filter1.png)

![](https://raw.githubusercontent.com/ntk148v/goignore/master/screenshots/filter2.png)

- Result, the current gitignore is updated.

![](https://raw.githubusercontent.com/ntk148v/goignore/master/screenshots/diff.png)
