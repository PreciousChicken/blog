---
title: "TiddlyWiki5, Raspberry Pi and Vim: A guide for the command line aficionado"
date: 2021-05-15T22:03:28+01:00
tags: ["tiddlywiki", "node", "Raspberry Pi", "bash", "vim"]
categories: ["PIM"]
description: ""
enableToc: true
draft: true
---

## Introduction

The practice of personal information management has always left me unsatisfied; a square hole in the puzzle of life that you just don't have a square peg for.  After looking into options for square pegs I've opted for a zettelkasten method implemented via a [TiddlyWiki](https://tiddlywiki.com/).  Wanting to do things my way, here were the requirements:

-  Host TiddlyWiki 5 on a Raspberry Pi.
-  Let other computers on my home network access that (including tablets and mobiles).
-  Be able to directly edit tiddlers (the basic unit of information in TiddlyWiki) using neovim, so I don't always have to go through a browser.

Version here.  On the desktop / laptop I'm using Manjaro Linux 21.0.5.

After the long-winded and unnecessary background I go through the steps as to how I achieved this.

## Long-winded and unnecessary background

This turned out to be more difficult than I thought.  The [vanilla node.js installation](https://github.com/Jermolene/TiddlyWiki5) of TiddlyWiki does not work particularly well with direct editing of tiddlers, i.e. editing the tiddlers manually won't be picked up in the browser without a manual restart of the application.  [OokTech](https://ooktech.com/) has an [executable](https://github.com/OokTech/TW5-BobEXE) that does allow direct editing - but this [doesn't work](https://github.com/OokTech/TW5-BobEXE/issues/18) on a Raspberry Pi.  Not to fear however as OokTech also have a [plugin](https://github.com/OokTech/TW5-Bob) which gives the same functionality.

So I used this for a number of months - however there was a snag.  Every so often I would edit a tiddler in the browser and, although appearing to save correctly, upon returning at a future point the update would have disappeared.  Weirdly looking at the tiddler directly in a text editor would show the updated version.  I had no idea what the problem was - and entirely possible it was nothing to do with the software, but how I had configured it.  Either way my rule of thumb when something breaks is to upgrade to the latest version.  So I updated both TiddlyWiki5 and the OokTech plugin; at which point it stopped working altogether.

A fresh install seemed the best option at this stage - but as I was running through the [instructions](https://github.com/OokTech/TW5-Bob#step-by-step-instructions-using-node), I wondered if there might be an easier way.  

## Enable Port 8080

As I've [blogged about previously](https://www.preciouschicken.com/blog/posts/node-tiddlywiki5-raspberry-pi-port-8080/), if you are using the default [Raspberry Pi OS](https://www.raspberrypi.org/software/) ports are shut down by default.  These therefore need to be opened by entering the following commands into the terminal:


```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 8080
```

The second line is only required if you are using ssh to remote into your Pi, if you aren't leave it out.  Hint - if you are using a keyboard connected to your Pi, then you aren't using ssh...

This installs [Uncomplicated Firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewall) onto your Pi, enables it and then allows the port that your TiddlyWiki will be listening on.

## Install Tiddlywiki5 and Nodemon

At the terminal type:

```bash
npm install tiddlywiki nodemon -g
```

The `g` flag installs the software globally, rather than in a particular folder.

## Initiate a wiki

We are now going to use TiddlyWiki to install a fresh wiki.  At the terminal:

```bash
tiddlywiki wiki --init server
```

This creates a directory called `wiki` which includes server-related components.

## Find your IP

Now we have to [find the IP address of your Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ip-address.md) and make a note of it.  For the purposes of this tutorial we are going to say it is `192.168.0.12`.

## Starting TiddlyWiki with nodemon

Ok, so now the fun bit, starting.  If you weren't bothered about editing the tiddlers in a text editor you could just simply go for:

```bash
tiddlywiki wiki --listen host=192.168.0.12
```

And if you go to [http://192.168.0.12:8080/](http://192.168.0.12:8080/) in your browser you should see a new wiki.

All well and good, however part of my requirements are to edit tiddlers in a text editor.  Using the above method to start would mean that edits do not get reflected in the browser - or not without manually restarting the node instance of TiddlyWiki.

This is where [nodemon](https://nodemon.io/) comes in.  Nodemon is software that watches a directory and restarts a node process any time there is a change in that directory.  It is commonly used for development purposes - if you are working on a JavaScript file you want that to be refreshed and reloaded every time you make a save - not every time you restart the server.  However it can be used to look for changes to tiddlers, rather than code.

To do so enter at the terminal this, intimidatingly long, command:

```bash
nodemon --delay 30 -e tid --ignore 'wiki/tiddlers/$*.tid' --watch wiki/tiddlers/ $NVM_BIN/tiddlywiki wiki --listen host=192.168.0.19
```
At this point you can simply walk away leaving the terminal open and running if you have been directly inputting commands into your Raspberry Pi.  If you are accessing your Pi remotely via ssh then close the terminal window itself (on Manjaro i3 this is done via the `mod-Shift-q` keypress, but your distro will likely be different) without exiting the running program.

So what does this do?  The flags / arguments to nodemon are as follows:

-  `--delay 30` ensures that after a change nodemon waits 30 seconds before restarting.  If this did not occur then the server would be constantly restarting.
-  `e tid` tells nodemon that it needs to look for changes in files ending in `tid`, that is in tiddlers.
-  `--ignore 'wiki/tiddlers/$*.tid'` ignores tiddlers that are generated by the system (which by convention start with `$`) and not the user.
-  `--watch wiki/tiddlers/` tells nodemon the folder to watch where changes will happen.
-  `$NVM_BIN/tiddlywiki wiki --listen host=192.168.0.19` lastly this is the previous command as at the start of the section, but as we aren't running the command via node we have to explicitly tell nodemon where to find the tiddlywiki executable (in *$NVM_BIN*).

## Mapping a drive

Connecting to Pi goes here


To make things a bit easier once we've decided where our tiddlers are we will set this path as an environment variable so we aren't having to retype the path again.  We do this by editing our `.bashrc` (or whatever shell we use - I use zsh so it is `.zshrc`) to include:

```bash
export TIDDLYWIKIPATH=$HOME/wiki/tiddlers/
```

## Vim plugin

I don't have that many plugins enabled in Neovim, but [vim-tiddlywiki](https://github.com/sukima/vim-tiddlywiki) is absolutely outstanding for managing tiddlers in Vim.  I don't use a plugin manager, so installing it into Neovim is as follows:

```bash
 git clone --depth 1 https://github.com/sukima/vim-tiddlywiki ~/.config/nvim/pack/sukima/start/vim-tiddlywiki
```

If you are using original Vim rather than Neovim, then the path above will need amending as Vim does not use the *.config* directory.

This plugin not only means that the tiddler format is recognised, but also allows you to create new tiddlers with the right metadata in place and jump to the other tiddlers using CamelCase links.

For full use you will also need to let the plugin know where you keep your tiddlers by adding the following line to your *~/.config/nvim/init.vim* or `~/.vimrc`, e.g.: 

```vimrc
let g:tiddlywiki_dir = '~/docs/notes/wiki' TODO
```

## Command line editing

To make the process buttery smooth I want to automate my workflow.  This being: read an article on the net, copy the title of the article and have neovim open a tiddler already named using CamelCase.  So for example, let's say I want to take some notes on Hayek's paper on open markets: [The use of knowledge in society](https://doi.org/10.1142/9789812701275_0025).  I want to open a terminal, type `tw The use of knowledge in society` and start editing a tiddler named TheUseOfKnowledgeInSociety.tid in neovim.

Create a file named *tw* wherever you keep user-specific executable files (so for me this is *~/.local/bin/tw*, but if you are using Ubuntu it will likely be *~/bin/tw*) and copy / paste the following:

```bash
#!/bin/bash

# Opens a new tiddlywiki tiddler named after arguments that follow commands
# Uses vim-tiddlywiki plugin to create metadata

# Replaces non-alphanumeric characters in arguments
arguments=$(echo $@ | sed "s/[’]//g")
arguments=$(echo $arguments | sed 's/[^a-zA-Z0-9]/ /g')

# CamelCase converts all arguments
tid_title=''
for word in $arguments
do
	# Changes argument to lowercase
	lowercasevar=${word,,}
	# Capitalises first character of argument
	sentencevar=${lowercasevar^}
	tid_title+="$sentencevar"
done

# Opens neovim with correct tiddler name
if [[ -f $TIDDLYWIKIPATH$tid_title'.tid' ]]
then
	# Updates metadata if tiddler exists
	nvim -c TiddlyWikiUpdateMetadata $TIDDLYWIKIPATH$tid_title'.tid'
else
	# Creates tiddler if not found
	nvim -c TiddlyWikiInitializeTemplate $TIDDLYWIKIPATH$tid_title'.tid'
fi
```

Then make this executable: 

```bash
chmod a+x ~/.local/bin/tw
```

On next restart of terminal `tw The use of knowledge in society` should create a new tiddler ready for us to type.  If the tiddler already exists then the shell script will recognise that and update the metadata instead.

The shell script does its best to cope with non-alphanumeric characters.  So apostrophes are deleted (*We're not really strangers* gets changed to *WereNotReallyStrangers*) while other non-alphanumerics characters are replaced by a space (so *Knee-deep in the Big Muddy* ends up as *KneeDeepInTheBigMuddy*).  I'm sure there will be edge cases which will not work out - feel free to comment below.

## A note on spawning

It is worth noting that nodemon can be a bit difficult to stop once it is started.  One option is to [find the process](https://linustechtips.com/topic/1049531-nodejs-how-to-stop-nodemon/) running and stop them individually:

> ps aux | grep -i nodemon, find out which process number nodemon is, then issue a kill -9 [process ID]

Or alternatively reboot the Pi.  That can be easier..
