---
title: "On Metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "keywords", "golang"]
categories: ["web dev"]
description: "How to automate metadata insertion within the HTML header using Hugo open-source static site generator"
draft: true
---

## The best being the enemy of the good

Before starting a blog I knew I was in considerable danger of spending a year researching blog content management software, not writing a word and eventually shelving the whole project.  As my inspiration for starting a blog was Guzey's [Why you should start a blog right now](https://guzey.com/personal/why-have-a-blog/) I figured a good heuristic was to use whatever he did.  [Hugo](https://gohugo.io/), an open-source static site generator, was therefore it.

The next choice was theme, another potential time sink, but [Cactus Plus](https://themes.gohugo.io/hugo-theme-cactus-plus/) won out.

## But meta

But like anything off-the-shelf there have been a number of annoyances and those mostly relate to how the theme handles metadata within the Header element.  Apparently much of this, like keywords, has been so badly abused by those looking to game Google, that it is now routinely ignored by search engines.  And even if it isn't I have a hunch that the semantic web will (is?) being overtaken by algorithms - i.e. why rely on the page author to sum up what your page is about, when the machine can do that for you.  However ultimately if there is a box to fill in, I like to do it.

After some time watching the excellent 

## Frontmatter example

As an example of what a typical frontmatter on this blog looks like, here is the frontmatter for this page:

```toml
---
title: "On Metadata in Hugo - or turning tags to keywords"
date: 2021-01-01T14:35:58Z
tags: ["hugo", "keywords", "golang"]
description: "How to automate metadata insertion within the HTML header using Hugo open-source static site generator"
draft: false
---
```

To get Hugo to default to this one needs to alter the file `archetypes\default.md`, editing the archetypes within your theme will make no difference (I'm not the only one to find this behaviour [slightly confusing](https://discourse.gohugo.io/t/hugo-doesnt-use-theme-archetypes/8382)).

### Tags to Keywords

The Cactus Plus theme does not make any attempt to complete the [keywords](https://html.spec.whatwg.org/multipage/semantics.html#meta-keywords) metadata attribute.  It does however make use of 'tags', which seen as you are already supplying seem close enough.  To add these then I added the following line to the file `themes/hugo-theme-cactus-plus/layouts/partials/head.html`:

```html
<meta name="keywords" content="{{ with .Params.tags }}{{ delimit . ", "}}{{ end }}" />
```

