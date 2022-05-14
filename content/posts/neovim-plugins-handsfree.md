---
title: "Look Ma, no Neovim plugin manager!"
date: 2022-05-14T07:47:32+01:00
tags: ["bash", "neovim"]
categories: ["SyOps"]
description: "How to manually install NeoVim plugins without a package manner"
enableToc: false
draft: true
---

## Handsfree

To manually install a [Neovim](https://neovim.io/) plugin without using a plug-in manager (using [Tim Pope's Commentary](https://github.com/tpope/vim-commentary) as an example) then at the terminal:

```bash
mkdir -p ~/.local/share/nvim/site/pack/tpope/start/commentary
git clone https://tpope.io/vim/commentary.git ~/.local/share/nvim/site/pack/tpope/start/commentary
nvim -u NONE -c "helptags ALL" -c q
```

## Automation

Which is great, but from time to time your plugins will get updated and you will need to pull fresh versions.  This can easily be achieved with a small script.  Create a file named ___~/.local/bin/vimpluginupdate___ and paste the following:

```bash
#!/bin/sh
NVMPTH='$HOME/.local/share/nvim/site/pack'
git -C "$NVMPTH/tpope/start/commentary" pull || git clone --depth 1 https://tpope.io/vim/commentary.git $NVMPTH/tpope/start/commentary
nvim -u NONE -c "helptags ALL" -c q
```

The rather long git line pulls the repository if it already exists, and if it does not exist (i.e. you haven't done the first step) then it clones a fresh one (using the ___depth___ option to just get the latest version and not previous changes).

Now make the file executable:

```bash
chmod u+x ~/.local/bin/vimpluginupdate
```

And whenever you want to update your plugins run:

```bash
vimpluginupdate
```

As you install more updates simply add new lines (ensuring that the helptags line remains the last line of the file).

## But I use vim?

All I've really done here is amended the [vim-commentary install section](https://github.com/tpope/vim-commentary#installation), so check that out for standard vim (NB - the main difference is the directory used).

## I demand an authoritative source 

See [:help packages](https://neovim.io/doc/user/repeat.html#packages) and [:help add-package](https://neovim.io/doc/user/usr_05.html#05.4)
