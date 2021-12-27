---
title: "Tentative steps into the Neovim API"
date: 2021-12-27T21:20:42Z
tags: ["neovim", "python"]
categories: ["tools"]
description: ""
enableToc: false
draft: true
---

Neovim's API is typically called through a Remote Procedure Call (RPC) using the [MessagePack-RPC](https://github.com/msgpack-rpc/msgpack-rpc/blob/master/spec.md) Specification.  This is a note for self as I take my first tentative steps exploring this.  I'm running Neovim v 0.6.0 on Manjaro 21.2.0 Qonos.

First we need to determine the servername that neovim has set on startup.  Fire up an instance of nvim with the command 

```bash
nvim -c "echo v:servername"
```

The `-c` [command option](https://neovim.io/doc/user/starting.html#-c) used above causes neovim to run a command on opening, in this case to print out the servername of the instance - resultantly we should see the servername of the instance in the statusline; it should look a little like: `/tmp/nvimt7lIuM/0`.

We can now use this address to command the neovim instance remotely.  Using the [example provided in the help](https://neovim.io/doc/user/api.html#API), create a file named `nvim-hw-api.py` and paste the following - with the path matching the string output above:

```py
from pynvim import attach
nvim = attach('socket', path='/tmp/nvimt7lIuM/0')
nvim.command('echo "hello world!"')
```

Now at the terminal run:

```bash
python3 nvim-hw-api.py
```

And if all works as forecast we should see `hello world` appear in the statusline of the neovim instance!
