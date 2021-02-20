---
title: "A bash script for creating image thumbnails using ImageMagick"
date: 2021-02-15T21:58:02Z
tags: ["Linux", "ImageMagick", "MarkDown", "Bash", "Hugo"]
categories: ["SysOp"]
description: "A bash script for creating image thumbnails"
enableToc: false
draft: false
---

Recently [Google Search Console](https://search.google.com/search-console/) has been alerting me to *mobile usability issues*, specifically *content wider than screen*:

![Google Search Console](https://www.preciouschicken.com/blog/images/usability_issues.png)

Digging further I found the problem was how my blog generating software, [Hugo](https://gohugo.io/), was linking to images.  Typically I insert images into markdown as so:

```md
[![Your message here](https://www.preciouschicken.com/blog/images/taste_of_react_your_message.png)](https://www.preciouschicken.com/blog/images/taste_of_react_your_message.png)
```

The image URL features twice: the first one displays the image on page resized to fit the theme; the second provides a 'clickable' link to the actual image should the user want to see the image in its full, actual size.

The issue that Google Search Console has is that I am providing an image that is too large and the browser is having to resize on the fly to fit the content.   It would be far better to provide a smaller image for display (at the size it appears in the browser), e.g. a thumbnail, while still providing the link to the bigger image.  Like so:

```md
[![Your message here](https://www.preciouschicken.com/blog/images/taste_of_react_your_message-thumb.png)](https://www.preciouschicken.com/blog/images/taste_of_react_your_message.png)
```

As I already had a folder full of images I decided to automate this with a bash script:

```bash
#! /bin/bash 
# Sets the desired thumbnail width
WIDTH=630
# Loops through all png files in current folder
for i in *.png
do
    # Stores the width of the current file
    iwidth=`identify -format "%w" $i`
    # Checks current filename does not end with '-thumb' and
    # file is greater than desired width
    if [[ $i != *-thumb.png ]] && [ $iwidth -gt $WIDTH ]
    then
        # Stores filename without extension
        filename=`basename -s .png $i`
        # Creates thumbnail and adds -thumb to end of new file
        convert -thumbnail ${WIDTH}x $i "$filename-thumb.png"
    fi
done
```

This loops through all the *png* files in the current folder, checks that they aren't a thumbnail (as indicated by a *-thumb.png* suffix), that they are bigger than the content and then resizes them using [ImageMagick](https://imagemagick.org/) - an image conversion tool which is installed by default on many Linux distributions.

To run this copy and paste the text above to a new file, e.g. *thumb_create*, make it executable and run it, e.g.:

```bash
chmod u+x thumb_create
./thumb_create
```

You should now be left with a folder full of original images, plus resized thumbnails as indicated by *-thumb.png*.  Apart from any original images which are smaller than the width you specified - these will not have a thumbnail created.  The script can be run as many times as you want, on subsequent runs it will ignore any files it has already generated (e.g. those ending *-thumb.png*).

NB - It could also probably be optimised: checking the filename prior to storing the width would likely speed things up.

