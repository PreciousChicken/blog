---
title: "On metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "keywords", "golang"]
categories: ["web dev"]
description: "How to automate metadata insertion within the HTML header using Hugo open-source static site generator"
draft: true
---

## The best being the enemy of the good

Before starting a blog I knew I was in considerable danger of spending a year researching blog content management software, not writing a word and eventually shelving the whole project.  As my inspiration for starting a blog was Guzey's [Why you should start a blog right now](https://guzey.com/personal/why-have-a-blog/) I figured a good heuristic was to use whatever he did.  [Hugo](https://gohugo.io/), an open-source static site generator, was therefore it.

The next choice was theme, another potential time sink, but [Cactus Plus](https://themes.gohugo.io/hugo-theme-cactus-plus/) won out.  In retrospect this might not have been the best choice as the maintainer stopped updating it in 2018.

## But meta

But like anything off-the-shelf there have been a number of annoyances and those mostly relate to how the theme handles metadata within the HTML [head element](https://html.spec.whatwg.org/multipage/semantics.html#the-head-element).  Whether this neglect of metadata is a specific problem within Cactus Plus, or whether it applies to many other Hugo themes, I am not aware.

Possibly Cactus Plus does not emphasise metadata, because many aspects of it (such as keywords) have been so badly abused by those looking to game Google, that it is now routinely ignored by search engines.  And even if it isn't ignored I have a hunch that the semantic web will (is?) being overtaken by algorithms - i.e. why rely on the page author to sum up what your page is about, when the machine can do that for you.  However ultimately I can't stand to see a round metadata hole without hammering a round metadata attribute in; a twisted metaphor if there ever there was but hopefully you get the point.

Although not having used Golang before, after watching Mike Dane's excellent [video tutorials](https://www.mikedane.com/static-site-generators/hugo/) on Hugo I felt ready to jump in.

## Front matter example

To ensure we have the correct metadata it has to be added by the author, much of this will be included in the [front matter](https://gohugo.io/content-management/front-matter/).  As an example of what a typical front matter on this blog looks like, here is the front matter for this page:

```toml
---
title: "On metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "keywords", "golang"]
description: "How to automate metadata insertion within the HTML header using Hugo open-source static site generator"
draft: false
---
```

To get Hugo to default to this template, so that creating a new post creates a blank front matter in the style above, one needs to alter the file `archetypes\default.md`.  Editing the archetypes within your theme will make no difference (I'm not the only one to find this behaviour [slightly confusing](https://discourse.gohugo.io/t/hugo-doesnt-use-theme-archetypes/8382/5?u=preciouschicken)).

## Partial template *head.html*

Within the Cactus Plus to ensure our metadata gets dragged from the front matter into the outputted HTML we need to alter the partial template found at *themes/hugo-theme-cactus-plus/layouts/partials/head.html*.  All the following attributes will necessitate an amendment to that file:

### Tags to Keywords

[Keywords](https://html.spec.whatwg.org/multipage/semantics.html#meta-keywords) are defined as:

> a set of comma-separated tokens, each of which is a keyword relevant to the page

The Cactus Plus theme not no attempt to complete the keywords metadata attribute.  It does however use 'tags', which seen as you are already supplying seem close enough.  To convert these to keywords I inserted the following line within the `<head>` element of the partial template *head.html*:

```html
<meta name="keywords" content="{{ with .Params.tags }}{{ delimit . ", "}}{{ end }}" />
```

Interestingly the [exact example](https://gohugo.io/functions/delimit/) of iterating through an array of tags is given in the Hugo documentation, but when used in anger it outputted a `Can't iterate over <nil>` error.  A [useful suggestion](https://discourse.gohugo.io/t/error-calling-delimit-cant-iterate-over-nil/23016/2?u=preciouschicken) in the community forum solved this:  `with` was required.

### Description

The metadata attribute [description](https://html.spec.whatwg.org/multipage/semantics.html#meta-description) is described as:

> a free-form string that describes the page. The value must be appropriate for use in a directory of pages, e.g. in a search engine.

Cactus Plus does insert a description into the metadata using this line in the partial template *head.html*:

```html
{{ with .Site.Params.description }}<meta name="description" content="{{ . }}">{{ end }}
```

However this inserts the description that one sets within the [configuration file](https://gohugo.io/getting-started/configuration/#configuration-file) - i.e. a description for the site as a whole (in my case "freelance software") rather than the description that relates to the page in question (in this instance Hugo metadata).  I therefore replaced this line with:

```html
<meta name="description" content="{{ if .Params.Description }}{{ .Params.Description }}{{ else if .Site.Params.Description }}{{ .Site.Params.Description }}{{ else }}Freelance software{{ end }}" /> 
```

This inserts the front matter description attribute if there is one, otherwise defaults to the configuration file description, and just to be belts and braces, if it can't find one of those goes for a default of "Freelance software".

### Canonical 

[RFC 6596 The Canonical Link Relation](https://tools.ietf.org/html/rfc6596) specifies: 

> The canonical link relation specifies the preferred IRI from resources with duplicative content.  Common implementations of the canonical link relation are to specify the preferred version of an IRI from duplicate pages

This is more serious.
