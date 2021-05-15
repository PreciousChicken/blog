---
title: "Tiddlywiki5 Raspberry Pi Guide"
date: 2021-05-15T22:03:28+01:00
tags: [""]
categories: [""]
description: ""
enableToc: true
draft: true
---

## Introduction

Oh boy this was difficult.  Here's what I wanted to do:

-  Host my brain dump on a Raspberry Pi.
-  Let other computers access that
-  Be able to directly edit tiddlers using neovim (or other text edit), so I don't have to go through web interface.

Should that be difficult?  




```bash
nodemon --delay 30 -e tid --ignore 'TW5nodemon2/tiddlers/$*.tid' --watch TW5nodemon2/tiddlers/ /home/pi/.nvm/versions/node/v14.11.0/bin/tiddlywiki TW5nodemon2 --listen host=192.168.0.19
```


Off and running?

https://linustechtips.com/topic/1049531-nodejs-how-to-stop-nodemon/

> ps aux | grep -i nodemon, find out which process number nodemon is, then issue a kill -9 [process ID]

