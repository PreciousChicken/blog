---
title: "Groff and the art of letter writing"
date: 2021-07-11T16:55:45+01:00
tags: ["groff"]
categories: ["office"]
description: ""
enableToc: true
draft: true
---

## Introduction

Although I've used LaTeX for academic writing before I thought I would give Groff a whirl for something I do from time to time: write formal letters.  Typically these are covering letters or letters of complaint or some such.  A typesetting system such as Groff appeals as the originals are stored in plain text which is memory efficient, useful for text searching within files and scores high for digital preservation (text files being easy to read as opposed to word processing formats).

## The letter

Open a file titled *complaint.me* and copy / paste:

```groff
.in 4i
12 High Street 
.br
Leicester
.br
LS1 1AB
.br
01234 567890
.br
another@example.org

8th July 2021

.in 0
SuperFibre Broadband
.br
Basildon
.br
PO Box 200

Dear Sir / Madam,
.pp
I am outraged and upset with your response\**
.(f
\** Reference C/A/O123 dated 1st July 2021
.)f
to my recent complaint.  To suggest that 
.i "my chickens"
pecking through 
.i "your broadband wire"
was an event outwith my contract with you is ludicrous.  
Any reputable communications company would ensure that their equipment had been through a testing process which ensures 
.b "fair wear and tear" 
caused by farmyard animals.
.pp
I insist that you release me from the remaining twelve months of my broadband contract immediately, and furthermore compensate me for the distress caused to my poultry.


Yours faithfully,


A N Other
```

## View on screen with Zathura

Assuming you have the [Zathura](https://pwmt.org/projects/zathura/) document viewer installed at the terminal type:

```bash
groff -me complaint.me | zathura -
```


## Output pdf file

If you would rather have a pdf file, in case you were emailing it for instance, then:

```bash
groff -me -T pdf complaint.me > complaint.pdf
```

## Output directly to print

```bash
groff -me complaint.me | lp
```

## Further References

- [How to format academic papers on Linux with groff -me](https://opensource.com/article/18/2/how-format-academic-papers-linux-groff-me).
- [-me Reference Manual](https://docs.freebsd.org/44doc/usd/20.meref/paper.pdf).
- [Writing Papers with NROFF using −me](https://docs.freebsd.org/44doc/usd/19.memacros/paper.pdf).
- [groff_me man page](https://man.cx/groff_me(7)).
- Luke Smith's groff [intro video](https://videos.lukesmith.xyz/videos/watch/6e8047a6-a940-481b-803c-6fc13fa22eb9).

