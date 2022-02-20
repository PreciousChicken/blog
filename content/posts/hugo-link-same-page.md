---
title: "Linking to the same page in Hugo"
date: 2022-02-20T14:58:24Z
tags: ["Hugo", "Markdown", "HTML"]
categories: ["Web development"]
description: "Linking to an element on the same page in Hugo framework"
enableToc: false
draft: false
---

## HTML

In HTML a [relative link to an element in the same page](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a#linking_to_an_element_on_the_same_page) looks like:

```html
<!-- <a> element links to the section below -->
<p><a href="#Section_further_down">
  Jump to the heading below
</a></p>

<!-- Heading to link to -->
<h2 id="Section_further_down">Section further down</h2>
```

## Hugo

In [Hugo](https://gohugo.io/) to achieve the same effect:

```markdown
<!-- <a> element links to the section below -->
[Jump to the heading below](#section-further-down)

<!-- Heading to link to -->
## Section further down
```
