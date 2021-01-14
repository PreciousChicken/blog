---
title: "Stacking Vercel, a GraphQL Apollo Server and React"
date: 2021-01-14T17:11:52Z
tags: ["GraphQL, React, Vercel, Apollo"]
categories: ["WebDev"]
description: "A worked example how to host a GraphQL Apollo Server on the Vercel platform with a React front end"
draft: true
---

## Introduction


All code can be found on github at: 

## Create-React-App

To get us started:

```bash
npx create-react-app vercel-apollo-server-react
cd vercel-apollo-server-react
npm i apollo-server-micro graphql
```

## GraphQL Server

Vercel looks for a folder called API to run serverless functions, so let us first create one of those:

```bash
mkdir api && cd api
```



