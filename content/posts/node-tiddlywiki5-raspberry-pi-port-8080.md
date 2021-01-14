---
enableToc: true
title: "Port 8080 fail using TiddlyWiki5 and Node on a Raspberry Pi"
date: 2020-09-18T15:10:21+01:00
tags: ["Node", "Raspberry Pi", "TiddlyWiki"]
categories: ["SysAdmin"]
description: "Fixing a Port 8080 error message whilst configuring TiddlyWiki5 and Node on a Raspberry Pi"
draft: false
---

## The Problem

There seem a ton of ways to get [TiddlyWiki](https://tiddlywiki.com/) running.  I thought I would try the [Node.js version](https://github.com/Jermolene/TiddlyWiki5) on a Raspberry Pi.  I wanted a set-up so I could leave the Pi online and access the TiddlyWiki on the rest of my home LAN.  After following the [instructions](https://tiddlywiki.com/static/Installing%2520TiddlyWiki%2520on%2520Node.js.html) I got to the last line:

```bash
tiddlywiki mynewwiki --listen
```

And I had TiddlyWiki running on the Pi, but couldn't access  on the rest of the LAN.  Which I wasn't expecting.  It turned out there were two problems:

-  Raspberry Pi comes with ports closed by default and
-  The default version of TiddlyWiki node needs a specific flag

Always more difficult when you have two problems, as you solve one and when it doesn't work, you think you've done something wrong; as opposed to realising you are half way there...  

Solution follows.

## Enable the port on Raspberry Pi

At the terminal on the Pi:

```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 8080
```

The second line is only required if you are using ssh to remote into your Pi, if you aren't leave it out.  Hint - if you are using a keyboard connected to your Pi, then you aren't using ssh...

This installs [Uncomplicated Firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewall) onto your Pi, enables it and then allows the port that your TiddlyWiki will be listening on.

## Set the host flag when running TiddlyWiki5

First we have to [find the IP address of your Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ip-address.md).  Let's assume you've done that and it is `192.168.0.12`.  You now have to start the TiddlyWiki with the following command:

```bash
tiddlywiki mynewwiki --listen host=192.168.0.12
```

You should now be able to access the TiddlyWiki using the address [http://192.168.0.12:8080](http://192.168.0.12:8080) from the rest of your network.  You won't be able to reach your TiddlyWiki from the external internet without further steps however (which suits my use case).

## Further reading

[Run your Node.js application on a headless Raspberry Pi](https://dev.to/bogdaaamn/run-your-nodejs-application-on-a-headless-raspberry-pi-4jnn) was pretty helpful.
