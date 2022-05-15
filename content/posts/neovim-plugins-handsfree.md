---
title: "Look Ma, no Neovim plugin manager!"
date: 2022-05-14T07:47:32+01:00
tags: ["bash", "neovim"]
categories: ["SyOps"]
description: "How to manually install NeoVim plugins without a package manner"
enableToc: false
draft: false
---

## Handsfree

To manually install a [Neovim](https://neovim.io/) plugin, for example [Tim Pope's Commentary](https://github.com/tpope/vim-commentary), without using a plug-in manager then at the terminal :

```bash
mkdir -p ~/.local/share/nvim/site/pack/tpope/start/commentary
git clone https://tpope.io/vim/commentary.git ~/.local/share/nvim/site/pack/tpope/start/commentary
```

The install path always follows the pattern of _~/.local/share/nvim/site/pack/foo/start/bar_, where the variables _foo_ and _bar_ can be whatever makes sense to you.  Commonly _foo_ is the name of the author (e.g. tpope) and _bar_ is the name of the plugin (e.g. commentary).

## Automation

Which is great, but from time to time your plugins will get updated and you will need to pull fresh versions.  This can easily be achieved on Linux systems with a small script.  Create a file named _~/.local/bin/vimpluginupdate_ and paste the following:

```bash
#!/bin/sh
NVMPTH='$HOME/.local/share/nvim/site/pack'
git -C "$NVMPTH/tpope/start/commentary" pull || git clone --depth 1 https://tpope.io/vim/commentary.git $NVMPTH/tpope/start/commentary
```

The rather long git line pulls the repository if it already exists, and if it does not exist (i.e. you haven't done the first step) then it clones a fresh one (using the _depth_ option to just get the latest version and not previous changes).

Now make the file executable:

```bash
chmod u+x ~/.local/bin/vimpluginupdate
```

And whenever you want to update your plugins run[^1]:

[^1]: Interestingly the _~/.local/bin/_ directory is not added to your _PATH_ [on a fresh install](https://askubuntu.com/a/1144235), so if the command does not work at first, log out then in again.

```bash
vimpluginupdate
```

As you install more updates simply add new lines.

## Helptags

To load any help documentation associated with the plugin then run `:helptags ALL` within Neovim when the plugin is active.  So for instance the vim-commentary plugin will be active when Neovim has open any file recognised as a programming language (i.e. any file with an extension _.js_, _.py_, etc).  In vim-commentary's case the help docs can then be accessed with `:help commentary`.


## But I use vim?

All I've really done here is amended the [vim-commentary install section](https://github.com/tpope/vim-commentary#installation), so check that out for standard vim (NB - the main difference is the directory used).

## I demand an authoritative source 

See [:help packages](https://neovim.io/doc/user/repeat.html#packages) and [:help add-package](https://neovim.io/doc/user/usr_05.html#05.4).

## Version control

This was tested corrected on Ubuntu Linux 22.04 LTS and Neovim v0.6.1.
