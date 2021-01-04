---
title: "On metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "html", "golang", "metadata"]
categories: ["web dev"]
description: "How to automate metadata insertion within the HTML head element using the Hugo open-source static site generator"
draft: false
---
## TL;DR

How (and why) to add the HTML metadata of keywords, description and canonical link to Hugo's Cactus Plus theme.  All code viewable on my [blog](https://github.com/PreciousChicken/blog) repository.

## The best being the enemy of the good

Before starting a blog I knew I was in considerable danger of spending a year researching blog content management software, not writing a word and eventually shelving the whole project.  As my inspiration for starting a blog was Guzey's [Why you should start a blog right now](https://guzey.com/personal/why-have-a-blog/) I figured a good heuristic was to use whatever he did.  [Hugo](https://gohugo.io/), an open-source static site generator, was therefore it.

The next choice was theme, another potential time sink, but [Cactus Plus](https://themes.gohugo.io/hugo-theme-cactus-plus/) won out (though in retrospect I wasn't to know it would stop being maintained in 2018).

## But meta

Like anything off-the-shelf some tweaking is required and for me that related to how Cactus Plus handles metadata within the HTML [head element](https://html.spec.whatwg.org/multipage/semantics.html#the-head-element).  Whether Cactus Plus' treatment of metadata is replicated more generally within the Hugo ecosystem, I don't know, but I do know that I preferred a more comprehensive approach.

Possibly Cactus Plus does not emphasise metadata, because many aspects of it (such as keywords) have been so badly abused by those looking to game Google, that it is now routinely ignored by search engines.  And even if it isn't ignored I have a hunch that the semantic web will (is?) being overtaken by algorithms - i.e. why rely on the page author to sum up what your page is about, when the machine can do a better job.  However ultimately I can't stand to see a round metadata hole without hammering a round metadata attribute in; a twisted metaphor if there ever there was but hopefully you get the point.

Although not having used Golang before, after watching Mike Dane's excellent [video tutorials](https://www.mikedane.com/static-site-generators/hugo/) on Hugo I felt ready to jump in.

## Front matter example

To ensure we have the correct metadata it has to be added by the author, much of this will be included in the [front matter](https://gohugo.io/content-management/front-matter/).  As an example of what a typical front matter on this blog looks like, here is some of the front matter for this page:

```toml
---
title: "On metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "html", "golang", "metadata"]
description: "How to automate metadata insertion within the HTML head element using the Hugo open-source static site generator"
draft: false
---
```

To get Hugo to default to this template, so that creating a new post produces blank variables ready for you to complete, one needs to alter the root file *archetypes\default.md*.  Editing the archetypes within your theme will make no difference - unless you have deleted your root *archetypes\default.md* (I'm not the only one to find this behaviour [slightly confusing](https://discourse.gohugo.io/t/hugo-doesnt-use-theme-archetypes/8382/5?u=preciouschicken)).

## Partial template *head.html*

Within Cactus Plus to ensure our metadata gets dragged from the front matter into the outputted HTML we need to alter the partial template found at *themes/hugo-theme-cactus-plus/layouts/partials/head.html*.  All the following attributes will necessitate an amendment to that file.  Other themes of course might place the HTML elsewhere.

Nodejs' [original](https://github.com/nodejh/hugo-theme-cactus-plus/blob/master/layouts/partials/head.html) *head.html* can be compared to my [final](https://github.com/PreciousChicken/blog/blob/master/themes/hugo-theme-cactus-plus/layouts/partials/head.html) on github.

### Tags to Keywords

[Keywords](https://html.spec.whatwg.org/multipage/semantics.html#meta-keywords) are defined as:

> a set of comma-separated tokens, each of which is a keyword relevant to the page

The Cactus Plus theme makes no attempt to complete the keywords metadata attribute.  It does however use 'tags', which seen as you are already supplying them seem close enough.  To convert these to keywords I inserted the following line within the `<head>` element of the partial template *head.html*:

```html
<meta name="keywords" content="{{ with .Params.tags }}{{ delimit . ", "}}{{ end }}" />
```

Interestingly the [exact example](https://gohugo.io/functions/delimit/) of iterating through an array of tags is given in the Hugo documentation, but when used in anger it outputted a `Can't iterate over <nil>` error.  Thankfully the [solution](https://discourse.gohugo.io/t/error-calling-delimit-cant-iterate-over-nil/23016/2?u=preciouschicken) was in the community forum:  `with` was required.

This should generate the following HTML:

```html
<meta name="keywords" content="hugo, html, golang, metadata" />
```

### Description

The metadata attribute [description](https://html.spec.whatwg.org/multipage/semantics.html#meta-description) is defined as:

> a free-form string that describes the page. The value must be appropriate for use in a directory of pages, e.g. in a search engine.

Cactus Plus does insert a description into the metadata using this line in the partial template *head.html*:

```html
{{ with .Site.Params.description }}<meta name="description" content="{{ . }}">{{ end }}
```

However this inserts the description set within the [configuration file](https://gohugo.io/getting-started/configuration/#configuration-file) - i.e. a description for the site as a whole (in my case "freelance software"), rather than the description that relates to the page in question (so for this page: "How to automate metadata insertion within the HTML head element using the Hugo open-source static site generator").  One can argue that in this case the behaviour of Cactus Plus is not aligned with the specification above which refers to the page rather than the site.  I therefore replaced this line with:

```html
<meta name="description" content="{{ if .Params.Description }}{{ .Params.Description }}{{ else if .Site.Params.Description }}{{ .Site.Params.Description }}{{ else }}Freelance software{{ end }}" /> 
```

This inserts the front matter description attribute if there is one, otherwise defaults to the configuration file description, and just to be belts and braces, if it can't find one of those goes for a default of "Freelance software".

This should generate the following HTML:

```html
<meta name="description" content="How to automate metadata insertion within the HTML head element using the Hugo open-source static site generator" />
```

### Canonical link

Cactus Plus does not insert a canonical link as metadata, which is described at [RFC 6596 The Canonical Link Relation](https://tools.ietf.org/html/rfc6596) as: 

> The canonical link relation specifies the preferred [URL] from resources with duplicative content.  Common implementations of the canonical link relation are to specify the preferred version of an [URL] from duplicate pages

In other words when you have two pages with identical content, this lets search engines know which of the two pages should be considered 'authoritative.'  This is particularly relevant to my circumstances: all my posts are first uploaded to my [blog](https://www.preciouschicken.com/blog/index.html) and then syndicated to [dev.to](https://dev.to/preciouschicken).  I therefore need a canonical link to [let Google know](https://developers.google.com/search/docs/advanced/crawling/consolidate-duplicate-urls) that the authoritative copy is the one on my site, not dev.to.

This to be fair is probably the correct behaviour on the part of Cactus Plus, it might be that your use case means the pages you are generating via Hugo are not the authoritative copy, or simply the only copy.  It just doesn't work for me.

The solution to this is easy, insert within the `<head>` element of *head.html* the following line:

```html
<link rel="canonical" href="{{ .Permalink }}" />
```

This should generate the following HTML:

```html
<link rel="canonical" href="https://preciouschicken.com/blog/posts/hugo-tags-to-keywords/" />
```

## Conclusion

I enjoy using Hugo as a content management system, it allows one to write from a standing start without getting bogged down in web design minutiae.  At the same time when change is required you are able to fully open the hood and tinker to your heart's content: as long as you are prepared to get to grips with Golang.  

Cactus Plus is also a lovely theme, that does 99% of what I need - just a pity it is no longer maintained.

As ever comments and feedback below welcomed.
