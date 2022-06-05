---
title: Scripting Basics
date: 05/21/22
keywords: [scripting, programming, commandline]
---

## Preface

Every computer user should know how to do basic scripting. It increases your productivity by an order of magnitude, and eliminates the need for many large programs. I would estimate that with just the unix core utilities and cURL you could script just about anything.

## Use the Google Shell Style Guide

Regardless of your opinion on Google, their shell style guide very will done. You should read it and apply it to every script you right. Having a consistant way of writing scripts will help make your scripts easier to read when you come back to them a year later. Also this style guide will help you to avoid some of the common gotcha's of shell scripting.

[Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)

## Use ShellCheck

From the ShellCheck github:

ShellCheck is a GPLv3 tool that gives warnings and suggestions for bash/sh shell scripts.

The goals of ShellCheck are:

- To point out clarify typical beginner's syntax issues that cause a shell to give cryptic error messages.
- To point out and clarify typical intermediate level semantic problems that cause a shell to behave strangly and counter-intuitively.
- To point out subtle caveats, corner cases and pitfalls that may cause an advanced user's otherwise working script to fail under future circumstances.

Every script you write should be run through ShellCheck. Not only will it catch a lot of bugs for you, it will also make you a better shell programmer.

You can get ShellCheck though your package manager or from [github](https://github.com/koalaman/shellcheck).

## Tutorials

Besides the Google Shell Style Guide there are really only two shell tutorials that I ever found useful, the `bash` and `dash` man pages. Pretty much every other tutorial is useless for learning.

## Should I use ZSH?

No. Bash is the default Linux shell. Eventually I will do a in depth video or writeup on the reasons why.
