---
title: "Writing a Simple Shell in C Sharp"
date: "2023-08-20T23:15:34+07:00"
tags: ["tech", "csharp"]
draft: true
comment: true
---

In this post, we will write a minimalistic shell for UNIX(-like) operating systems in C# programming language. I create this for learning C# purpose.

Since its purpose demonstration (not feature completeness or even fitness for causual use), it has many limitations, including:

- Commands must be on a single line.
- Arguments must be separated by whitespace.
- No quoting arguments or escaping whitespace.
- No piping or redirection.

## 1. What is a shell?

In computing, a [shell](<https://en.wikipedia.org/wiki/Shell_(computing)>) is a computer program that exposes an operating system's services to a human user or other programs. In general, operating system shells use either a command-line interface (CLI) or graphical user interface (GUI), depending on a computer's role and particular operation. It is named a shell because it is the outermost layer around the operating system.

Some examples:

- Bash
- Zsh
- Gnome Shell

In this article, I describe _a simple text-based, non-graphical shell_ with the basic functionality: give an input command and receive the output of this command.

```shell
# Input
$ ls

# Output
Documents   Downloads   Desktop
...
```
