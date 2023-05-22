---
title: "Vegaration: Visualising continuous integration using Github actions and vega-lite"
date: 2023-05-22T18:50:14+01:00
tags: ["github actions", "vega-lite", "jest"]
categories: ["testing"]
description: "Worked example of visualising continuous integration using Github actions and vega-lite"
enableToc: true
draft: true
---

## Introduction

This is a worked example which aims to visualise the software development practice of continuous integration (CI).  It uses [GitHub actions](https://github.com/features/actions) to manage the CI workflow and [vega-lite](https://vega.github.io/vega-lite/) as the visualisation specification.  Erm, what?  In other words every time someone commits to a repository on GitHub we can capture elements associated with their commit (e.g. what tests passed, what tests failed) and then present this using a range of visualisations such as bar charts etc.

All code is contained in the vegaration repo TODO and the visualisations can be viewed on their own GitHub page.

This was written using Manjaro Linux 22.1.1, node v16.13.2 and npm v8.3.2.

## Create the GitHub repo

