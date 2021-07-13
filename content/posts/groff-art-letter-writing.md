---
title: "Groff and the art of letter writing"
date: 2021-07-11T16:55:45+01:00
tags: ["groff", "linux"]
categories: ["text processing"]
description: ""
enableToc: true
draft: true
---

## Introduction

Although I've blogged previously about using [LaTeX for academic writing](https://www.preciouschicken.com/blog/posts/neovim-latex-zathura-in-perfect-harmony/) I wanted to try Groff out for the less complicated task of letter writing.  Typically these are covering letters or letters of complaint or some such.  A typesetting system such as Groff appeals as the originals are stored in plain text which is memory efficient, useful for text searching within files and scores high for digital preservation (text files being easy to read as opposed to word processing formats).

## The letter

Open a file titled *complaint.me* and copy / paste:

```groff
.wh 2c hd \" Top margin (header) set to 2cm
.wh -2c fo \" Bottom margin (footer) set to 2cm
.po 2c \" Right margin set to 2cm
.ll 17c \" Line length set to 17 cm
.in 13c \" Address block indented 13 cm
12 High Street 
.br \" Line break
Leicester
.br
LS1 1AB
.br
01234 567890
.br
another@example.org
.sp \" Spaces one line
8th July 2021
.sp
.in 0 \" Address block indent removed
SuperFibre Broadband
.br
Basildon
.br
PO Box 200
.sp
Dear Sir / Madam,
.pp \" Paragrapgh start
I am outraged and upset with your response\** \" Places footnote
.(f \" Footnote start
\** Reference C/A/O123 dated 1st July 2021.
.)f \" Footnote end
to my recent complaint.  To suggest that 
.i "my chickens" \" Italics
pecking through 
.i "your broadband wire"
was an event outwith my contract with you is ludicrous.  
Any reputable communications company would ensure that their equipment had been through a testing process which ensures 
.b "fair wear and tear"  \" Bold
caused by farmyard animals.
.pp
I insist that you release me from the remaining twelve months of my broadband contract immediately, and furthermore compensate me for the distress caused to my poultry.
.sp 2 \" Spaces two lines
Yours faithfully,
.sp 2 
A N Other
```

## View on screen with Zathura

Assuming you have the [Zathura](https://pwmt.org/projects/zathura/) document viewer installed at the terminal type:

```bash
groff -me complaint.me | zathura -
```

This command pipes postscript into Zathura's standard input with the following result:

[![A letter of complaint written in Groff](https://www.preciouschicken.com/blog/images/groff-letter-thumb.png)](https://www.preciouschicken.com/blog/images/groff-letter.png)

## Output pdf file

If you would rather have a pdf file, then you add the *-T* flag which indicates you want the output as something other than the default postscript:

```bash
groff -me -T pdf complaint.me > complaint.pdf
```

## Output directly to print

Assuming a [default printer has been set](https://www.mattcutts.com/blog/change-default-printer-linux-firefox/), then we can also send the postcript directly to the printer (with no internmediate pdf needing to be produced):

```bash
groff -me complaint.me | lp
```

## Me, me, me?

As groff is very low-level then macro facilities are grouped together into packages to allow routine operations (such as footnotes) to be done efficiently.  *Me* is one of these packages; although [far from the only option](https://www.stephenlindholm.com/groff_macros.html).  The *-me* flag on the command line means that groff uses this package and we save it with a .me extension.

## Further References

- [How to format academic papers on Linux with groff -me](https://opensource.com/article/18/2/how-format-academic-papers-linux-groff-me).
- [Writing Papers with NROFF using −me](https://docs.freebsd.org/44doc/usd/19.memacros/paper.pdf).
- [-me Reference Manual](https://docs.freebsd.org/44doc/usd/20.meref/paper.pdf).
- [groff_me man page](https://man.cx/groff_me(7)).
- Luke Smith's [groff/troff: MUH MINIMALIST Documents](https://videos.lukesmith.xyz/videos/watch/6e8047a6-a940-481b-803c-6fc13fa22eb9) introduction video.
- [The GNU Troff Manual](https://www.gnu.org/software/groff/manual/groff.html)

