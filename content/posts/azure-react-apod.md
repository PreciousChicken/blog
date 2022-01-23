---
title: "Azure React Apod"
date: 2022-01-21T22:16:06Z
tags: ["React", "Azure Web App", "Node"]
categories: ["Web development"]
description: "A demonstration of NASA's Astronomy Picture of the Dat (APOD) using the React framework, hosted as an Azure web app."
enableToc: true
draft: true
---


## Introduction

Hosting a React framework Single Page Application as an Web App on the Microsoft Azure can be an extremely frustrating process, for a number of reasons.  To start with despite React being (rightly or wrongly) the [most popular web framework](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/), there are no specific instructions within Azure documentation for how one might deploy a React SPA as an Azure Web App.  Far worse however is that the choice of how operating system when creating a web app (Windows or Linux) is an essential criteria (in short it is [very difficult to use create-react-app to deploy a Web App on a base Linux OS](https://github.com/MicrosoftDocs/azure-docs/issues/32572#issuecomment-637832128)) yet this appears almost entirely undocumented.

Create a react application:

```bash
npx create-react-app azure-react-apod
```

Install az cli:



Upload to azure:

```bash
az webapp up --sku F1 --name azure-react-apod
```

## Further reading:

* [Host a web application with Azure App Service](https://docs.microsoft.com/en-gb/learn/modules/host-a-web-app-with-azure-app-service/?WT.mc_id=azureportalcard_Service_App%20Services_-inproduct-azureportal)
