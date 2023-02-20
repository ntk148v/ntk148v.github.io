---
title: "Rename Files in Linux"
date: 2021-12-16T10:22:05+07:00
tags: ["linux", "tech"]
toc: true
comments: true
draft: false
---

- Rename a single file with `mv`. Just a basic thing.
- Rename multiple files with `mv`.

```bash
# Rename files with suffix .yaml to yml
for f in *.yaml; do mv -- "$f" "${f%.yaml}.yml" done
```

- Rename multiple files with `rename`.

```bash
# Install rename command
# Ubuntu/Debian-derived distros
sudo apt install rename
# RedHat-derived distros
sudo yum install prename
# The follow examples are performed in Ubuntu/Debian-derived distros
rename 's/.yaml/.yml/' *.yaml
# Replace all occurrences of "prev_" with "next_"
rename 's/prev_/next_' *.c
# Delete part of a filename
rename 's/next_//' *.c
# Limit changes to specific parts of filenames
# Only change the files that start with "paramater"
rename 's/^param/parameter/' *.c
# Search with grouping
# Replace all occurrences of "string" and "strong"
rename 's/(str|stro)ng/strength' *.c
# Use translations with rename
# Force filenames to uppercase
rename 'y/a-z/A-Z' *.py
# More?
man rename
```
