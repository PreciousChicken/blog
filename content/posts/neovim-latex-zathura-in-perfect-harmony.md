---
title: "Neovim, LaTeX and Zathura in perfect harmony"
date: 2020-05-23T18:09:07+01:00
tags: ["neovim", "laTeX", "zathura", "vimscript"]
categories: ["Unix workbench"]
description: "Vimscript to get Zathura to display LaTeX pdfs without leaving Neovim"
draft: false
---

### Introduction

[Zathura](https://pwmt.org/projects/zathura/) is my pdf viewer of choice: it is minimalist and has vim key bindings by default.  I use it a lot when I'm writing TeX files in [Neovim](https://neovim.io), partly as you can open it without leaving Neovim.  And when you recompile the TeX file (I tend to use a makefile for this which I run from within vim) then it updates on the fly.  As I'm using the i3 window tiling manager as part of Regolith OS this results in my screen reconfiguring itself just how I want it:

[![Neovim and Zathura side by side](https://www.preciouschicken.com/blog/images/nvim_zathura.png)](https://www.preciouschicken.com/blog/images/nvim_zathura.png)

Although calling Zathura isn't a big deal from inside Neovim, I thought I'd shave off a couple of seconds each time and reduce this to a key press by taking half a day to write my first ever vimscript.  You may say false economy, I say a good way to procrastinate when you have a deadline due.

So it only takes me a quarter of a day if I decide to do something like this again, here are the steps.  For reference I'm using Ubuntu 18.04.4 LTS on the [Regolith](https://regolith-linux.org) desktop environment, TeX 3.14159265, Neovim v0.4.3 and Zathura 0.3.8 (with the plugin pdf-poppler 0.2.8).

### Create a ftplugin file for TeX files

At the terminal type:

`nvim ~/.local/share/nvim/site/ftplugin/tex.vim`

This creates a file which Neovim will read when, and only when, you are editing TeX files; there's no reason you can't just place the code that follows in your .vimrc, but I think this approach is neater.  This also assumes that your folder structure follows mine, which it probably should do if you are using Linux and haven't messed with it too much.

I'm led to believe if you are using plain vim rather than neovim this should be:

`vim ~/.vim/ftplugin/tex.vim`

### Add the vimscript

Copy-paste the following vimscript into the file you've just created:

```vim
function! ZathuraOpenPdf()
	let curFile = substitute(bufname("%"), "tex", "pdf", "")
	execute "silent !zathura " curFile "&"
endfunction

nnoremap <C-p> :call ZathuraOpenPdf()<CR>
```

This finds the name of the current file from the buffer, chops the "tex" off the end, replaces it with "pdf" and then runs Zathura in a new window whenever you press the \<Control-p\> key combination.

### Conclusion

This was my first time trying vimscript and given how notorious vim's [learning curve](https://thoughtbot.com/blog/the-vim-learning-curve-is-a-myth) is, there are bound to be better / neater / more formally correct ways to have written this - feel free to comment below if this is the case.

### Added bonus - Makefile

In case you are interested below are the contents of my Makefile, where _mydocument.tex_ is the name of the relevant TeX document (the `biber` line refers to the bibliography software, if you aren't using this it can be just shortened to one turn of pdflatex):

```make
default: build

build:
	pdflatex mydocument.tex
	biber mydocument
	pdflatex mydocument.tex
	pdflatex mydocument.tex

clean:
	-rm *.aux 
	-rm *.log 
	-rm *.lof 
	-rm *.bbl 
	-rm *.blg 
	-rm *.lot 
	-rm *.out 
	-rm *.toc 
	-rm *.bcf 
	-rm *.run.xml 
	-rm *.blx.bib 
```

If this is copy-pasted into a file called _Makefile_, then running `:!make` in Neovim will update your pdf, and in turn Zathura.  Actually you don't need the `!` and can just use `:make` but for some reason this doesn't return me back to the TeX file when I've finished, so I just insert the bang.

### Further reference

- [Learn vimscript the hard way](https://learnvimscriptthehardway.stevelosh.com)
- [Vim scripting cheatsheet](https://devhints.io/vimscript)
- [Scripting the Vim editor, Part 1: Variables, values, and expressions](https://developer.ibm.com/articles/l-vim-script-1/)
