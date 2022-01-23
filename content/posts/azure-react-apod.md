---
title: "Azure React Apod"
date: 2022-01-21T22:16:06Z
tags: ["React", "Azure Web App", "Node"]
categories: ["Web development"]
description: "A demonstration of NASA's Astronomy Picture of the Day (APOD) using the React framework, hosted as an Azure web app."
enableToc: true
draft: true
---


## Introduction

Hosting a React framework Single Page Application as an Web App on the Microsoft Azure can be an extremely frustrating process, for a number of reasons.  To start with despite React being (rightly or wrongly) the [most popular web framework](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/), there are no specific instructions within Azure documentation for how one might deploy a React SPA as an Azure Web App.  Far worse however is that the choice of how operating system when creating a web app (Windows or Linux) is an essential criteria (in short it is [very difficult to use create-react-app to deploy a Web App on a base Linux OS](https://github.com/MicrosoftDocs/azure-docs/issues/32572#issuecomment-637832128)) yet this appears almost entirely undocumented.

The nub of the problem is that once you've created a React app the actually part that gets shown to the public is produced by running `npm run build` and then serving the resulting web pages found in the *build* folder to the web - everything outside the build folder is essentially for development, not for showing to the public.  Often this detail is taken care of behind the scenes, so for instance the [Vercel](https://vercel.com/) platform knows what a React SPA is and does the donkey work for you (see [Stacking Vercel, a GraphQL Apollo Server and React](https://www.preciouschicken.com/blog/posts/vercel-apollo-server-react/) for more on Vercel and React).  Azure does not apparently realise any of this so you have to upload the contents of the *build* folder to [Azure's */site/wwwroot)* directory](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure.  However this only works if you have chosen Windows as your OS when creating the Web App, if you have chosen Linux it simply does not work.  No explanation given, and a couple of hours consigned to trying to figure out where you have gone wrong.

And CI/CD via Github workflow is not exactly a joy either; none of the explanations online appear from start to finish without modification.  As an aside the access the Azure App Service wants of your Github account is onerous too (you can't restrict access to private repos - which, er, are meant to be private).

This therefore is going to be a run through of how to set this up relatively start to finish on a Linux machine.  It assumes you have an Azure account, a GitHub account and node / npm installed.  Development is being done on Manjaro Linux 21.2.1, Node 16.13.2, npm 8.3.2.

## 1.  Set up an Azure Web App

First we need to set up a Web App on Azure.  From the Azure portal select **Create a Resource** and then **Web App**.  Go through the various options until you are faced with a confirmation screen that looks like:


![Azure create Web App](https://www.preciouschicken.com/blog/images/azure-react-apod-create_web_app.png)

The key point here is that the **Operating System** is given as **Windows**, if **Linux** is selected it will not work.  You might want to check that you are on the Free plan too (**F1**) as opposed to the paid for (**B1**) which is the default.  We've named the app *react-apod* but name it whatever you wish.

## 2.  Use create-react-app to generate the React SPA

On our local machine from the terminal enter:

```bash
npx create-react-app azure-react-apod
```

A React SPA will now be created using [create-react-app](https://create-react-app.dev/) tool.

Once installed change directory into the App:

```bash
cd azure-react-apod
```

## 3.  Make some changes to the React App

TODO.

## 4.  Create a Github repo and push the local React app

We are going to enable continuous deployment (CD) on this project via Github.  Therefore create a new repo in Github (without adding README, .gitignore or licence) and then push the project to it.  Digital Ocean features a good tutorial on [pushing an existing project to GitHub](https://www.digitalocean.com/community/tutorials/how-to-push-an-existing-project-to-github).

As a reminder the basic terminal commands (once you've created the new empty repo), changing the github PreciousChicken url to your own, are:

```bash
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:PreciousChicken/azure-react-apod.git
git branch -M main
git push -u origin main
```

## 5.  Download publish profile from Azure

Navigate to your Web App within the Azure portal and select the ***Get publish profile*** option, as shown below, to download a file named *react-apod.PublishSettings* to your local machine:

[![Azure portal get publish profile](https://www.preciouschicken.com/blog/images/azure-react-apod_get_publish_profile-thumb.png)](https://www.preciouschicken.com/blog/images/azure-react-apod_get_publish_profile.png)

## 6.  Add publish profile as a secret to GitHub repo

Following the GitHub guide [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) create a new secret with the name ***AZURE_WEBAPP_PUBLISH_PROFILE*** and copy and paste the contents of the publish profile you downloaded earlier.  To make this easier on Linux you can copy a file into the clipboard from the terminal (assuming you have [xsel](http://www.vergenet.net/~conrad/software/xsel/) installed) like so:

```bash
xsel -b < ~/Downloads/azure-react-apod.PublishSettings
```

This secret will allow GitHub to access Azure portal in order to deploy your web app on your behalf.

## 7.  Add a workflow to Github

We now need to tell GitHub how to deploy your web app, we do this by adding a workflow.

Therefore create a new file, named *azure-webapps-node.yml*,  located in a new GitHub workflow directory in your local repo:

```bash
mkdir -p  ./.github/workflows/ && touch ./.github/workflows/azure-webapps-node.yml
```

Then copy and paste the following into it (changing the name of the *env* variables according to the comments):

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AZURE_WEBAPP_NAME: react-apod    # change to your application's name in Azure
  AZURE_WEBAPP_PACKAGE_PATH: '/home/runner/work/azure-react-apod/azure-react-apod'  # change both instances of azure-react-apod to the name of your repo 
  NODE_VERSION: '16.x'                # set this to the node version to use if not 16

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: npm install, build, and test
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present

    - name: Add process.json
      run: |
        echo '{ "script": "serve", "env": { "PM2_SERVE_SPA": "true", "PM2_SERVE_HOMEPAGE": "index.html" } }' >> ./build/process.json
    - name: Upload artifact for deployment job
      uses: actions/upload-artifact@v2
      with:
        name: node-app
        path: ./build

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Development'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
    - name: Download artifact from build job
      uses: actions/download-artifact@v2
      with:
        name: node-app


    - name: 'Deploy to Azure WebApp'
      id: deploy-to-webapp 
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```



Install az cli:



Upload to azure:

```bash
az webapp up --sku F1 --name azure-react-apod
```

## Further reading:

* [Host a web application with Azure App Service](https://docs.microsoft.com/en-gb/learn/modules/host-a-web-app-with-azure-app-service/?WT.mc_id=azureportalcard_Service_App%20Services_-inproduct-azureportal)
* [https://websitebeaver.com/deploy-create-react-app-to-azure-app-services](https://websitebeaver.com/deploy-create-react-app-to-azure-app-services) - This was a good attempt at getting ninety percent there, but did not work out all the way.
