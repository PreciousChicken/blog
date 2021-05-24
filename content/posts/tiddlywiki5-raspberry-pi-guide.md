---
title: "The full guide to running TiddlyWiki5 on a Raspberry Pi"
date: 2021-05-15T22:03:28+01:00
tags: ["tiddlywiki", "node", "Raspberry Pi"]
categories: ["PIM"]
description: ""
enableToc: true
draft: true
---

## Introduction

The practice of personal information management has always left me unsatisfied; a square hole in the puzzle of life that you just don't have a square peg for.  After some time into square peg research and decided to try out the zettelkasten method using a [TiddlyWiki](https://tiddlywiki.com/).  Wanting to do things my way, here were the requirements:

-  Host TiddlyWiki 5 on a Raspberry Pi.
-  Let other computers on my home network access that (including tablets and mobiles).
-  Be able to directly edit tiddlers using neovim (or other text edit), so I don't always have to go through a browser.

Version here.  

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

Ok, so now the fun bit, starting.  The simple route to doing this is:

```bash
tiddlywiki wiki --listen host=192.168.0.12
```

If you go to [http://192.168.0.12:8080/](http://192.168.0.12:8080/) in your browser you should see a new wiki.

All well and good, however part of my requirements are to edit tiddlers in a text editor.  Using the above method to start would mean that edits do not get reflected in the browser (or not without manually restarting the node instance of TiddlyWiki).

This is where nodemon comes in.  Nodemon is software that watches a directory and restarts a node process any time there is a change in that directory.  It is commonly used for development purposes - if you are working on a JavaScript file you want that to be refreshed and reloaded every time you make a save - not every time you restart the server.  However it can be used to look for changes to tiddlers, rather than code.

To do so at the terminal enter this, intimidatingly long, command:

```bash
nodemon --delay 30 -e tid --ignore 'wiki/tiddlers/$*.tid' --watch wiki/tiddlers/ $NVM_BIN/tiddlywiki wiki --listen host=192.168.0.19
```




I now just close the terminal!

## Off and running?

https://linustechtips.com/topic/1049531-nodejs-how-to-stop-nodemon/

> ps aux | grep -i nodemon, find out which process number nodemon is, then issue a kill -9 [process ID]

