---
title: "Groff and the art of letter writing"
date: 2021-07-11T16:55:45+01:00
tags: ["groff", "linux"]
categories: ["text processing"]
description: "Worked example using groff text processing on Linux for letter writing."
enableToc: true
draft: false
---

## Introduction

Although I've blogged previously about using [LaTeX for academic writing](https://www.preciouschicken.com/blog/posts/neovim-latex-zathura-in-perfect-harmony/) I wanted to try [groff](https://www.gnu.org/software/groff/) out for the less complicated task of letter writing - groff being similar to LaTeX but [far less popular](https://unix.stackexchange.com/questions/89625/is-troff-groff-relevant-anymore/). Using a typesetting system such as groff instead of word processing software has several advantages: saving documents in plain text is memory efficient, easily searchable and scores highly for digital preservation (proprietary word processing file formats tending to eventually obsolesce).

Below is a worked example on how to use groff to produce a formal letter on Linux.  I'm using groff version 1.22.4, Manjaro Linux 21.0.7 and Zathura 0.4.7.

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
.pp \" Paragraph start
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
groff -me -dpaper=a4 complaint.me | zathura -
```

This command pipes postscript into Zathura's standard input with the following result:

[![A letter of complaint written in groff](https://www.preciouschicken.com/blog/images/groff-letter-thumb.png)](https://www.preciouschicken.com/blog/images/groff-letter.png)

## Output directly to print

Assuming a [default printer has been set](https://www.mattcutts.com/blog/change-default-printer-linux-firefox/), then we can also send the postcript directly to the printer:

```bash
groff -me -dpaper=a4 complaint.me | lp
```

As with the previous command this instructs groff to format for A4 sized paper.

## Output pdf file

The previous commands avoided the use of PDF all together, but if you would rather save in that format, then adding the *-T* flag indicates you want the output something other than default postscript:

```bash
groff -me -dpaper=a4 -T pdf complaint.me > complaint.pdf
```

## Me, me, me?

As groff is very low-level then macro facilities are grouped together into packages to allow routine operations (such as footnotes) to be done efficiently.  *Me* is one of these packages, although [far from the only option](https://www.stephenlindholm.com/groff_macros.html); in fact a new macro package, *[-mk](http://ankarstrom.se/~john/mk.html)*, was released this very month.  The *-me* flag on the command line informs groff this notation is being used, likewise the [recommended file extension](https://man7.org/linux/man-pages/man5/groff_filenames.5.html) to use is *.me*.

## Conclusion

I guess the one obvious question is why use groff at all, as opposed to LaTeX?    Possibly because there is something satisfying about learning modern uses for an application whose lineage dates back to 1964.   Or maybe just, to paraphrase George Mallory, because it is there?  Thoughts, observations?  Feel free to add below.  

## Further References

- [How to format academic papers on Linux with groff -me](https://opensource.com/article/18/2/how-format-academic-papers-linux-groff-me).  Gentle introduction to groff and the *-me* package.
- [Writing Papers with NROFF using −me](https://docs.freebsd.org/44doc/usd/19.memacros/paper.pdf).  Introductory paper written by Eric P. Allman, author of the *-me* package.
- [*-me* Reference Manual](https://docs.freebsd.org/44doc/usd/20.meref/paper.pdf).  Full reference for *-me*, again written by Allman.
- [groff_me man page](https://man7.org/linux/man-pages/man7/groff_me.7.html).  Linux groff_me(7) Miscellaneous Information Manual.
-  [groff/troff: MUH MINIMALIST Documents](https://videos.lukesmith.xyz/videos/watch/6e8047a6-a940-481b-803c-6fc13fa22eb9) Luke Smith's introductory video to groff, though uses the -ms package rather than *-me*.
- [The GNU Troff Manual](https://www.gnu.org/software/groff/manual/groff.html).  The definitive groff manual.  However only one sentence on the *-me* package.

