---
title: "A no-tears guide to adding references in Groff"
date: 2023-01-17T13:10:15Z
tags: ["groff", "linux"]
categories: ["typesetting"]
description: "Worked example of adding citations / references to Groff PDF documents using Refer and a bibliography file."
enableToc: true
draft: false
---

## Introduction

Groff is a decades-old typesetting system present on most Linux distributions which uses an application called refer to add citations.  I found adding citations using this mechanism a challenge, therefore as an aide-memoire this is a minimal worked example of adding references to a Groff document with the refer program.  I'm using GNU Groff version 1.22.4, GNU refer (groff) version 1.22.4 on Manjaro Linux 22.0.0.  I'm using the [ms macro](https://www.gnu.org/software/groff/manual/html_node/ms.html#ms), but the basic principles should apply to most macros.  All text examples can be found in the [associated github repo](https://github.com/PreciousChicken/no-tears-reference-groff).

## Creating documents

First create the document we are placing the references into, so create a file named *notears.ms* and paste the following:

```groff
.R1 \" Citation commands start
database notears.bib # Path to bibliography file
accumulate # Collates References at end of documentation
move-punctuation # Ensures that citation appears before full-stop.
label "(A.n|Q) ', ' (D.+yD.y%a*D.-y)" # Actual format of citation (e.g. (Author, Date)
bracket-label " (" ) "; " # Bracket
no-label-in-reference # Does not display full citation (e.g. Author, Date) within References
.R2 \" Citation commands end
.ds FAM H \" Sets font family
.TL
A no-tears guide to adding references in Groff
.AU
PreciousChicken
.AB
The abstract of a no-tears guide to adding references in Groff, see https://preciouschicken.com/blog/posts/no-tears-references-groff/ for more detail.
.AE
.PP
Here is a reference to a story about brain cells playing ping-pong
.[
shepherd22
.]
in my first paragraph.
.PP
And another reference to an AI story
.[
callaway20
.]
in my second paragraph.
.PP
Now we are on the final paragraph and I have my third
.[
nao22tax
.]
and then last reference,
.[
nao22bbc
.]
both from the NAO and both published in 2022.  I've also added a footnote for comparison purposes\**.
.FS \" Footnote start
This is the only footnote.
.FE \" Footnote end.
```

The section starting `.R1` and ending `.R2` is where commands are added as to how the citations will displayed.  These commands could also be entered on the command line rather than in the document text - but I prefer the latter as that keeps it all together if you are trying to recreate the document later.  For a full list of the applicable commands see [man(1) refer](https://man7.org/linux/man-pages/man1/refer.1.html).  The point where the citations are introduced into the text are marked by the opening `.[` and closing `.]`, with the keyword of the citation inbetween (e.g. `callaway20`).

Keen readers may note the comments format for the text between `.R1` and `.R2` follows a different format (comments begin with `#` not `\"`) as this section is intended for the preprocessor refer and not groff itself.

Now we need to create the bibliography file that this document points to, so in the same folder[^1] create another file named *notears.bib* and paste:

[^1]: The bibliography can be in another folder, in which case include the full path in the `database` command.

```groff
%K shepherd22
%A Shepherd, T.
%B London
%D 2022
%T Scientists teach brain cells to play video game Pong
%J The Guardian
%O Available at: https://www.theguardian.com/australia-news/2022/oct/13/scientists-teach-brain-cells-to-play-virtual-pong [Accessed 14 Oct 22]

%K callaway20 
%A Callaway, E.
%B London
%D 2022
%T 'It will change everything': Deepmind's AI makes gigantic leap in solving protein structures.
%J Nature News
%O Available at: https://www.nature.com/articles/d41586-020-03348-4 [Accessed 8 Sep 22]

%K nao22tax
%Q National Audit Office
%C London
%D 2022
%T Managing tax compliance following the pandemic
%O Available at: https://www.nao.org.uk/reports/managing-tax-compliance-following-the-pandemic/ [Accessed 17 Jan 23]

%K nao22bbc
%Q National Audit Office
%C London
%D 2022
%T A digital BBC
%O Available at: https://www.nao.org.uk/reports/a-digital-bbc/ [Accessed 17 Jan 23]
```

Each reference is separated by a blank line, with each field of the reference on a new line.  Each reference line starts with the code for the field, so for instance `%C` is city of publication.  The keyword we referred to in *notears.ms* is given on the `%K` line.  Again for the guide see [man(1) refer](https://man7.org/linux/man-pages/man1/refer.1.html).   The mom macro has also [introduced some fields](https://schaffter.ca/mom/momdoc/refer.html#fields-quick) which were not in the original (for instance `%l` as translator).

## Producing the PDF

We now have enough to produce a document with references.  There are a number of commands[^2] you can use to produce the final file, but we'll go for:

```sh
groff -R -m ms -T pdf notears.ms > notears.pdf
```

The `-R` option here being critical as it informs groff that there are references and to use the refer preprocesser before producing the final document.

If you now open the resulting `notears.pdf` with your pdf viewer of choice you should see:

[![No tears pdf screenshot](https://www.preciouschicken.com/blog/images/no-tears-thumb.png)](https://www.preciouschicken.com/blog/images/no-tears.png)

Of note is that the National Audit Office citations are disambiguated - i.e. 2022a and 2022b.  This is as a result of the rather cryptic `D.+yD.y%a*D.-y` code in the `label` field of the document - and is copied straight from [man(1) refer](https://man7.org/linux/man-pages/man1/refer.1.html).

## Conclusion & further references

If you've found this useful please leave a comment below or star the [github repo](https://github.com/PreciousChicken/no-tears-reference-groff).  You might also be interested in a previous post I wrote on [producing correspondence letters](https://www.preciouschicken.com/blog/posts/groff-art-letter-writing/) in groff or if you are a vim user my vim plugin: [vim-groff-viewer](https://preciouschicken.com/software/vim-groff-viewer/).

Apart from the [man(1) refer](https://man7.org/linux/man-pages/man1/refer.1.html) page there is not a huge amount of concise online guidance on including references within groff, however a useful video is Luke Smith's [Your Brain Using REFER to do your bibliographies automatically in groff/troff](https://videos.lukesmith.xyz/w/5ANbTYv7cgF69FhpAkVBwi).

[^2]: You could also use `refer notears.ms | groff -m ms -T pdf > notears.pdf` or `grog -T pdf --run notears.ms > notears.pdf` or `pdfroff -R -m ms notears.ms --pdf-output=notears.pdf` all of which would produce the same result.  Though interestingly in my testing I noticed that pdfroff used a more up to date version of the PDF format: v1.7 compared to the others using v1.4.
