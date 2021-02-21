---
title: "Creating a tasks todo list in TiddlyWiki"
date: 2021-02-21T09:11:46Z
tags: ["tiddlywiki", "wikitext"]
categories: ["PIM"]
description: "Instructions and WikiText required to create a tiddler that tracks tasks, heavily based on Francis Meetze's youtube tutorial"
enableToc: true
draft: false
---

## Introduction

This is entirely based on Francis Meetze's youtube tutorial, [Tracking Tasks in TiddlyWiki](https://www.youtube.com/watch?v=mzoMhKx0j8g), which although is fantastic does not provide the actual text to copy and paste.  All credit is his, mistakes are mine.

If you haven't seen the video, [TiddlyWiki](https://tiddlywiki.com) is a note taking tool (at its simplest); and the following steps allows you to create a special type of tiddler (e.g. note) which allows you to track tasks or todos.

This was written using version 5.1.21 of TiddlyWiki.

## Create a *Todo* tiddler

Firstly create a new tiddler - call it *Todo*, or something that makes sense to you.  Add a *Tasks* tag.  This is where a list of active tasks will appear.

Edit this tiddler by inserting the following WikiText:

```wikitext
<$list filter="[!has[draft.of]tag[task]!tag[done]sort[created]]">

<$checkbox tag="done"> <$link to={{!!title}}>
<$view field="title"/>
</$link>
</$checkbox>

</$list>
```

I've edited this slightly from Meetze's example as I don't want the time/date I created the task to feature - it doesn't add value for me.


## Create a *Completed* tiddler

Now we need a tiddler to list tasks which have been marked as done.  Create a tiddler called *Completed* and add a *Tasks* tag.

Add the following WikiText:

```wikitext
<$list filter="[!has[draft.of]tag[task]tag[done]sort[created]]">

<$checkbox tag="done">~~<$link to={{!!title}}>
<$view field="created" format="date" template="DDth mmm hh:mm"/> - <$view field="title"/>
</$link>~~</$checkbox>

</$list>
```

## Test it works

Create a tiddler entitled *TakeTheBinsOut*, give it a tag of *task.*  Once saved this should be listed on the *Todo* tiddler.  Checking the tickbox next to it should automagically cause it to be removed from *Todo* and appear on *Completed*.

## Add to default tiddlers

I've set my TiddlyWiki up so that my open tasks are the first visible tiddler on opening.  To do this open the Control Panel (the icon that looks like a cog) and add *Todo* to the first position in the *Default tiddlers* option.

## What, no screenshots?

True, I do normally, but in this case you can go and look at the [video](https://www.youtube.com/watch?v=mzoMhKx0j8g).  If you have any other feedback - feel free to add in comments.

## Addendum

As is always the case, in the course of writing a post I always later stumble across more on the subject.  Now I find the WikiText to complete the above is already within the TiddlyWiki documentation at their [TaskManagementExample](https://tiddlywiki.com/#TaskManagementExample) tiddler.  Their code is more concise, and they also provide a further [draggable example](https://tiddlywiki.com/#TaskManagementExample%20(Draggable)), so take a look if you want to dig deeper.
