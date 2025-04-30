---
title: "Dashed React Component"
date: 2025-03-27T18:53:59Z
tags: [""]
categories: [""]
description: ""
enableToc: true
draft: true
---

## Introduction

[Dash](https://dash.plotly.com/) is a low-code Python framework for data visualisation.  As Dash is built upon React, a JavaScript framework, you can use React to build your own components and Plotly provides both a JavaScript and TypeScript template to do so.  This is a worked example of how to use the latter of these templates to create your own React components for use in Dash.  There are minor differences when you use the JavaScript version

Version information.  You'll need node.

## Installation and set up

Some Python packages are needed to be installed prior to start, I'm using [pipx](https://pipx.pypa.io/stable/) to install these cleanly, but alternatively you might wish to create your own virtual environment instead:

```bash
pipx install cookiecutter virtualenv
```
If you are not using the TypeScript template then you will also need to install virtualenv should you wish to install dependencies during build:

```bash
pipx install cookiecutter virtualenv
```
With that out of the way we can install the template itself:

```bash
cookiecutter gh:plotly/dash-typescript-component-template
```

This will ask a series of questions, so you are using the same terms as this tutorial ensure the ***[1/11] project_name*** is ***Dashed Tutorial***, ***[3/11] component_name*** is ***pcCard*** and ***[11/11] publish_on_npm*** is ***n***.  Accept the default for the remainder of the questions.

As cookiecutter creates a directory named after the project name we chose, next enter the template directory with:

```bash
cd dashed_tutorial
```

If you have used the JavaScript template you will likely have download the Python dependencies on build, if you used the TypeScript one there's some additional steps:

```bash
python -m venv venv
```
then activate it:

```bash
source venv/bin/activate
```
and install the required Python packages:

```bash
pip install -r requirements.txt
```

Now we turn our attention to the React side of things and install the npm pacakges required.  This will install the Dash components that already exist within the template `package.json` file as well as the React libraries we are going to import for our demonstration.

```bash
npm install @mui/material @emotion/react @emotion/styled
```
and lastly we want to build the example React component already in the template:


```bash
npm run build
```

## Initial spin up



## Further reading

- Useful forum thread [Introducing TypeScript Dash component generation](https://community.plotly.com/t/introducing-typescript-dash-component-generation/61988), complete with nice example.
