---
title: "Where Do I Put AppImages?"
date: 2019-12-31T11:05:50Z
draft: true
---

**TL;DR:** I put my AppImages in */opt*.

The AppImages documentation originally (I believe) did not make a reccomendation as to which folder AppImages should be stored in.  The [AppImage FAQ](https://docs.appimage.org/user-guide/faq.html#question-where-do-i-store-my-appimages) now however recommends:

> If you don’t want to leave them in $HOME/Downloads, then $HOME/Applications is a good choice.

What's wrong with this?  Well firstly am I actually going to leave them in *$HOME/Downloads*?  No.  If you are anything like me *Downloads* is full of random dross that is accumulated as you browse the net, kind of like digital lint.  Antyhing worth preserving gets filed, anything left in there is assumed to be delete-worthy.  Which admittedly gets done very infrequently, but enough that it is not a permanent storage place.

So if this doesn't work why not create *$HOME/Applications* and use this?  Although this makes more sense than *Downloads* it still doesn't fit.  As my choice of OS is [Regolith](https://regolith-linux.org) I access applications via [Rofi](https://github.com/davatorium/rofi) - i.e. I open an application by pressing a key combination and then typing its name.  Even if I was using stock Ubuntu then I would presumably use the dock to access application icons.  I therefore have no need, pretty much ever, to navigate to a folder, find the right application and then double-click on it to open.  I have even less need to do this, if this folder does not contain all of my applications but only a small-subset of them (i.e. the ones I have installed using an AppImage) - which is the case here.  Now my understanding of the *$HOME* folder is that it is meant to actually contain things which the user cares about and wants to interact with - otherwise we might as well put everything in *$HOME* and not bother with the rest of the filesytem.  So why put them in a folder in *$HOME* if we are never actually going to want to access them; why don't we put them somewhere out of the way with the rest of the Linux gubbins?

So I'm far from an expert on Linux gubbins however the [description of */opt*](https://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/opt.html) seems to fit the bill:

> This directory is reserved for all the software and add-on packages that are not part of the default installation. For example, StarOffice, Kylix, Netscape Communicator and WordPerfect packages are normally found here. To comply with the FSSTND, all third party applications should be installed in this directory.

As this is outside of the *$HOME* directory the AppImage has to be moved as the superuser, e.g.:

```bash
sudo mv ~/Downloads/Subsurface\*.AppImage /opt/
chmod a+x Subsurface\*.AppImage
./Subsurface\*.AppImage
```
