---
title: "Port 8080 blues using Tiddlywiki5 and Node on a Raspberry Pi"
date: 2020-09-18T15:10:21+01:00
draft: false
---

## The Problem

There seem a ton of ways to get [TiddlyWiki](https://tiddlywiki.com/) running.  I thought I would try the [Node.js version](https://github.com/Jermolene/TiddlyWiki5) on a Raspberry Pi.  I wanted a set up so I could leave the Pi online and access the TiddlyWiki on the rest of my home LAN.  After following the instructions I could access the TiddlyWiki on the Pi, but not on the rest of the LAN.  It turned out there were two problems:

-  Raspberry Pi comes with ports closed by default and
-  The default version of TiddlyWiki node needs a specific flag

Phew.  Always more difficult when you have two problems, as you solve one and when it doesn't work, you think you've done something wrong, as opposed to being half way there...  

Anyway solution below.

## Enable the port on Raspberry Pi

At the terminal on the Pi:

```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 8080
```

The second line is only required if you are using ssh to remote into your Pi.  If you aren't you can leave it out.

This installs [Uncomplicated Firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewall) onto your Pi, enables it and then allows the Port that your TiddlyWiki will be listening on.

## Set the host flag when running TiddlyWiki5

First we have to find the IP address of your Raspberry Pi.  


