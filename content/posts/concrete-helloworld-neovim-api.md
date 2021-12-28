---
title: "A concrete Python 'Hello World' with the Neovim API"
date: 2021-12-27T21:20:42Z
tags: ["neovim", "python"]
categories: ["tools"]
description: "A worked example of a Hello World demo using the Neovim API and Python."
enableToc: false
draft: false
---

Neovim's API is typically called through a Remote Procedure Call (RPC) using the [MessagePack-RPC](https://github.com/msgpack-rpc/msgpack-rpc/blob/master/spec.md) specification.  The [neovim documentation](https://neovim.io/doc/user/api.html#rpc-connecting) provides a 'Hello World' example of how to interface with the Neovim API; this post is a worked example of how to implement this - more of a note to self than anything else.  I'm running Neovim v 0.6.0 on Manjaro 21.2.0 Qonos linux.

First we need to determine the [servername](https://neovim.io/doc/user/eval.html#v:servername) that Neovim has set on startup.  Fire up an instance of nvim from the terminal with the command:

```bash
nvim -c "echo v:servername"
```

The `-c` [command option](https://neovim.io/doc/user/starting.html#-c) used above causes neovim to run a command on opening, in this case to print out the servername of the instance.  Resultantly we should see the servername of the instance in the statusline; it should look a little like: `/tmp/nvimt7lIuM/0`.

We can now use this address to command the neovim instance remotely.  Create a file named `nvim-hw-api.py` and paste the following - altering the path to match the servername we found above:

```py
from pynvim import attach
nvim = attach('socket', path='/tmp/nvimt7lIuM/0')
nvim.command('echo "hello world!"')
```

At the terminal run (assuming [Python3 is installed](https://wiki.python.org/moin/BeginnersGuide/Download)):

```bash
python3 nvim-hw-api.py
```

And if all works as forecast we should see `hello world!` appear in the statusline of the neovim instance!
