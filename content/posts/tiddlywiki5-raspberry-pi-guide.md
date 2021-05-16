---
title: "The full guide to running Tiddlywiki5 on a Raspberry Pi"
date: 2021-05-15T22:03:28+01:00
tags: ["tiddlywiki", "node"]
categories: ["PIM"]
description: ""
enableToc: true
draft: true
---

## Introduction

Having played around with various personal information management implementations for what seems most of my life, having been impressed with the zettelkasten method I decided to try this out [TiddlyWiki](https://tiddlywiki.com/).  These were the requirements:

-  Host TiddlyWiki 5 on a Raspberry Pi.
-  Let other computers on my home network access that (including tablets and mobiles).
-  Be able to directly edit tiddlers using neovim (or other text edit), so I don't always have to go through a browser.

## Background

This turned out to be more difficult than I thought.  The [vanilla node.js installation](https://github.com/Jermolene/TiddlyWiki5) of TiddlyWiki does not work particularly well with direct editing of tiddlers, i.e. editing the tiddlers manually won't be picked up in the browser without a manual restart of the application.  [OokTech](https://ooktech.com/) has an [executable](https://github.com/OokTech/TW5-BobEXE) that does allow direct editing - but this [doesn't work](https://github.com/OokTech/TW5-BobEXE/issues/18) on a Raspberry Pi.  Not to fear however as OokTech also have a [plugin](https://github.com/OokTech/TW5-Bob) which gives the same functionality.

So I used this for a number of months - however there was a snag.  Every so often I would edit a tiddler in the browser and, although appearing to save correctly, when returning to the tiddler in the browser the update would have disappeared.  Weirdly looking at the tiddler directly in a text editor would show the updated version.  I had no idea what the problem was, but generally speaking my first rule of thumb when something breaks is to make sure you are running the most uptodate version.  So I updated both TiddlyWiki5 and the OokTech plugin; at which point it stopped working altogether.

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

## Starting TiddlyWiki with nodemon

```bash
nodemon --delay 30 -e tid --ignore 'TW5nodemon2/tiddlers/$*.tid' --watch TW5nodemon2/tiddlers/ /home/pi/.nvm/versions/node/v14.11.0/bin/tiddlywiki TW5nodemon2 --listen host=192.168.0.19
```


## Off and running?

https://linustechtips.com/topic/1049531-nodejs-how-to-stop-nodemon/

> ps aux | grep -i nodemon, find out which process number nodemon is, then issue a kill -9 [process ID]

