---
title: "Updating Node and npm using Node Version Manager"
date: 2021-03-28T09:20:02+01:00
tags: ["node", "bash"]
categories: ["SysAdmin"]
description: "Updating Node and npm using Node Version Manager on the command line"
enableToc: false
draft: false
---

[Node Version Manager](https://github.com/nvm-sh/nvm) is a "POSIX-compliant bash script to manage multiple active node.js versions."  It is a very useful tool for taming Node.js which can throw up some [strange problems](https://stackoverflow.com/q/16151018/6333825) when you install it in the [standard fashion](https://nodejs.org/en/download/).

However I only ever use two commands, which I can never remember off the top of my head, so here they are:

## Update Node.js to latest Long Term Support version

```bash
nvm install --lts
```

## Update npm

```bash
nvm install-latest-npm
```

## Where am I again?

And to confirm which versions you are using:

```bash
node --version && npm --version
```
