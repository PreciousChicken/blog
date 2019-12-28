---
title: "Configuring Oni as a C / C++ IDE on Ubuntu 18.04"
date: 2018-09-15T16:39:04Z
tags: ["Vim", "C Programming Language", "Oni", "IDE"]
draft: false
---

**Update Dec 19**: Whereas [Oni](https://www.onivim.io/) was freely downloadable, there since appears to have been a change in project direction.  The IDE is now called Onivim 2 and there is a fee to pre-order.  No idea if you can still install the version I used below, but I'm not deleting the post as it might be useful in other contexts.

---

[K&R's Second Edition of the C Programming Language](https://amzn.to/2Omcno5) (\*affiliate link) is not an easy read.  To tame it I thought I might use [Oni](https://www.onivim.io/) to do some of the heavy lifting, which is an IDE with [Neovim](https://neovim.io/) as a back end.  It looked pretty and was meant to work out of the box.  I however spent an age getting it to do linting got C, so below are my struggles: 

---

### 1.  **Install Oni**. 

So this is fairly straightforward. Head to their [downloads](https://www.onivim.io/Download) page and install the deb package via Ubuntu Software. Easy enough but opening the application will get an error:"Uh oh! Unable to launch Neovim... Neovim v0.2.1 is required to run Oni."

### 2.  **Install Neovim**. 

There are a number of ways to do this, the default one is to install an appimage. I'm hesitant about appimages; a good idea but not entirely sure how you are meant to update them. Instead elsewhere on the [downloads](https://github.com/neovim/neovim/wiki/Installing-Neovim) page there is an Ubuntu Personal Package Archive (PPA). Unfortunately, at time of writing, the [stable](https://launchpad.net/~neovim-ppa/+archive/ubuntu/stable) PPA isn't at the current version, while the [unstable](https://launchpad.net/~neovim-ppa/+archive/ubuntu/unstable) version [breaks Oni](https://github.com/onivim/oni/issues/2580). To install the stable version of neovim:

`sudo add-apt-repository ppa:neovim-ppa/stable`  
`sudo apt update`  
`sudo apt install neovim`

This will get you a working version of Oni, but will not get you IDE features for C or C++.

### 3.  **Install clang**. 
    
This is where things get more complicated. Although Oni does work out of the box for languages such as JavaScript and HTML, it doesn't for C or C++. Instead their [language-support guidance](http://github.com/onivim/oni/wiki/Language-support#cc) tells you that you need `clangd`. `Clangd` leverages `clang` and is an implementation of the Language Server Protocol [intended to](http://clang.llvm.org/extra/clangd.html) "provide language 'smartness' features like code completion... for... C/C++ editors." There are PPAs for both in Ubuntu that can be installed with:

`sudo apt install clang`  
`sudo apt install clang-tools`

Although you might think this would do it - it doesn't. When Oni initialises it expects to find `clangd` within the `/usr/bin/` directory, however the name of the file the PPA installs is different: `clangd-6.0` (referring to the version number). To solve this a symlink can be created referencing the expected file:

`sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-6.0 1000`

---

And wallah, Oni works for C and C++, almost out of the box... Interestingly once I had got it working it slowed to a crawl when you where looking at code longer than a hundred or so lines, so I abandoned it and went back to straight neovim.
