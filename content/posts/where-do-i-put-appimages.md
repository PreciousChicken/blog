---
title: "Where Do I Put AppImages?"
date: 2019-12-31T11:05:50Z
tags: ["AppImage", "Linux"]
keywords: ["AppImage", "Linux"] 
categories: ["Unix workbench"]
description: "A discussion on the most logical place to put AppImages within the Linux Filesystem Hierarchy"
draft: false
---

### TL;DR

I put my AppImages in */opt*.

### The Official Recommendation

The [AppImage FAQ](https://docs.appimage.org/user-guide/faq.html#question-where-do-i-store-my-appimages) recommends the following for storing AppImages:

> If you don’t want to leave them in $HOME/Downloads, then $HOME/Applications is a good choice.

### Why not leave them in $HOME/Downloads?

What's wrong with this?  Can't I just leave them in *$HOME/Downloads*?  No.  If you are anything like me *Downloads* is full of random accumulated dross, kind of like digital lint.  Anything worth preserving gets filed, anything left is assumed to be delete-worthy.  Which admittedly gets done very infrequently, but enough that it is not a permanent storage place.

### So $HOME/Applications, then?

So why not create and use *$HOME/Applications*?  Although this makes more sense than *Downloads*, it still doesn't fit.  As my choice of OS is [Regolith](https://regolith-linux.org) I access applications via [Rofi](https://github.com/davatorium/rofi) - i.e. I open an application by key combination and then typing its name.  Even if I was using stock Ubuntu then I would presumably use the dock to access application icons.  I therefore have no need, pretty much ever, to navigate to a folder, find the right application and then double-click on it to open.  I have even less need to do this when the folder does not contain all of my applications but only a subset of them (i.e. the ones I have installed using an AppImage) - which is the case here.  Now my understanding of the *$HOME* folder is that it is meant to actually contain things which the user cares about and wants to interact with - otherwise we might as well put everything in *$HOME* and not bother with the rest of the filesytem.  This [criticism has also been raised with Snap applications](https://bugs.launchpad.net/ubuntu/+source/snapd/+bug/1575053), which follows a similar delivery format.  So why put them in a folder in *$HOME* if we are never actually going to want to access them; why don't we put them somewhere out of the way with the rest of the Linux gubbins?

### The answer is /opt

So I'm far from an expert on Linux gubbins however the [description of */opt*](https://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/opt.html) seems to fit the bill:

> This directory is reserved for all the software and add-on packages that are not part of the default installation. For example, StarOffice, Kylix, Netscape Communicator and WordPerfect packages are normally found here. To comply with the FSSTND, all third party applications should be installed in this directory.

As this is outside of the *$HOME* directory the AppImage has to be moved as the superuser, e.g. (using [neovim.appimage](https://github.com/neovim/neovim/releases) as an example):

```bash
sudo mv ~/Downloads/nvim.appimage /opt
cd /opt
chmod a+x nvim.appimage && ./nvim.appimage
```

### System integration and appimaged

Using the */opt* directory also works with [appimaged](https://github.com/AppImage/appimaged) which is a daemon that looks for AppImages and registers them with the system (i.e. ensures their icons display etc).

### Cons

A potential disadvantage of using */opt* is that all users of the system can access the application, rather than just the current user.  Personally I think that is a plus.

I guess another issue with */opt* is you might forgot that is where you put all your AppImages - there's an easy answer to that: write a blog post...

Another option given in the AppImage FAQ is *$HOME/.local/bin* and *$HOME/bin* but as the FAQ itself states these are "are useful mainly for CLI tools," and most of my AppImages aren't in this category (though I guess nothing wrong in itself with putting these in a folder intended mostly for CLI tools).

---

A highly opinionated rant; */opt* works for me, use wherever works for you.  It's also worth adding the designers of AppImages where probably more interested in technical challenges and getting kudos from Linus Torvalds than these minor housekeeping points.  Comment below if you think I'm being dangerously outrageous.
