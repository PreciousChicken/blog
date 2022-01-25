---
title: "Photos of the Cosmos: Deploying a React Web App on the Microsoft Azure Cloud"
date: 2022-01-21T22:16:06Z
tags: ["React", "Azure Web App", "Node"]
categories: ["Web development"]
description: "A demonstration of NASA's Astronomy Picture of the Day (APOD) using the React framework, hosted as an Azure web app."
enableToc: true
draft: true
---


## Introduction

Hosting a React framework Single Page Application as an Web App on the Microsoft Azure can be an extremely frustrating process, for a number of reasons.  To start with despite React being (rightly or wrongly) the [most popular web framework](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/), there are no specific instructions within the Microsoft documentation for how one might deploy a React SPA as an Azure Web App.  Far worse however is that the choice of how operating system when creating a web app (Windows or Linux) is an essential criteria - in short it is [very difficult to use create-react-app to deploy a Web App on a base Linux OS](https://github.com/MicrosoftDocs/azure-docs/issues/32572#issuecomment-637832128)) - yet this is both unintuitive and seemingly undocumented.

The nub of the problem is that once you've created a React app the actually part that gets shown to the public is produced by running `npm run build` and then serving the resulting web pages found in the *build* folder to the web - everything outside the build folder is essentially for development and not for public view.  Often this detail is taken care of behind the scenes, so for instance the [Vercel](https://vercel.com/) platform knows what a React SPA is and does the donkey work for you (see [Stacking Vercel, a GraphQL Apollo Server and React](https://www.preciouschicken.com/blog/posts/vercel-apollo-server-react/) for more on Vercel and React).  Azure does not apparently realise any of this so you have to upload the contents of the *build* folder to [Azure's */site/wwwroot* directory](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure).  However this only works if you have chosen Windows as your OS when creating the Web App, if you have chosen Linux it simply does not work.  No explanation given, and a couple of hours consigned to trying to figure out where you have gone wrong.

And CI/CD via Github workflow is not exactly a joy either; none of the explanations online worked for me from start to finish without inexplicably failing half way through.  As an aside the access the Azure App Service, which I'm not using in this demo, wants of your Github account is onerous too (you can't restrict access to private repos - which, er, are meant to be private).

This therefore is going to be a run through of how to deploy a React Single Page Application to an Azure web app as start to finish as I can make it on a Linux machine.  GitHub will be used for continuous integration / continuous development.  It assumes you have an Azure account, a GitHub account and node / npm installed.  Some common command line tools are also used: sed and curl.  Development is being done on Manjaro Linux 21.2.1, Node 16.13.2, npm 8.3.2.  To give the demo more flavour, I am going to use the [NASA Astronomy Picture of the Day API](https://github.com/nasa/apod-api) to make a more fully featured app - you don't have to do this though, Section 3 can be omitted if you just want to deploy the create-react-app template to Azure.

## 1.  Set up an Azure Web App

First we need to set up a Web App on Azure.  From the Azure portal select **Create a Resource** and then **Web App**.  Go through the various options until you are faced with a confirmation screen that looks like:


![Azure create Web App](https://www.preciouschicken.com/blog/images/azure-react-apod-create_web_app.png)

The key point here is that the **Operating System** is given as **Windows**, if **Linux** is selected it will not work.  You might want to check that you are on the Free plan too (**F1**) as opposed to the paid for (**B1**) which is the default.  We've named the app *nasa-apod-picker* but you will need to give it a different name to prevent conflict.

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

If you are just interested in uploading any old React Single Page Application to Azure then you can safely skip this section and leave the default create-react-app settings in place.

However that does seems a bit dull.  The (somewhate detailed) instructions below will let us create a rather lovely app where we can pick NASA's Astronomy Photo of the Day by date.

### 3a.  Change app title and description

Change the App title and description to something more meaningful (we could do this by editing the *public/index.html* file, but using [sed](https://www.gnu.org/software/sed/) from the terminal is quicker:

```bash
sed -i 's/React App/NASA APOD Picker/g' public/index.html
sed -i 's/Web site created using create-react-app/Date picker for NASAs Astronomy Picture of the Day/g' public/index.html
```
### 3b.  Replace logos

The favicon and logos are currently the default create-react-app one; so we'll replace that by using [curl](https://curl.se/) to overwrite them with suitably space-themed ones[^1] (feel free to use whatever files you want, or simply leave the c-r-a one in place):

[^1]: Photo of moon by [OldakQuill, Public domain, via Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Moon.jpg).

```bash
curl https://www.preciouschicken.com/blog/images/azure-react-apod_favicon.ico > public/favicon.ico
curl https://www.preciouschicken.com/blog/images/azure-react-apod_logo192.png > public/logo192.png
curl https://www.preciouschicken.com/blog/images/azure-react-apod_logo512.png > public/logo512.png
```
### 3c.  Replace App.js file

The meat of the change, erase the contents of the *src/App.js* file and replace with:

```jsx
import './App.css';
import 'isomorphic-fetch';
import { useState } from "react";
import TextField from '@mui/material/TextField';
import AdapterDateFns from '@mui/lab/AdapterDateFns';
import LocalizationProvider from '@mui/lab/LocalizationProvider';
import enLocale from 'date-fns/locale/en-GB';
import DatePicker from '@mui/lab/DatePicker';
import { format, isWithinInterval } from 'date-fns';

function App() {
  const [apod, setApod] = useState([]);
  const [userDate, setUserdate] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [hasError, setHasError] = useState(false);
  const [errorState, setErrorstate] = useState(null);

  // Earliest photo in APOD
  const apodEarliest = new Date(1995,5,16); 
  // Latest photo in APOD, ie today
  // Hours set to midnight to ensure user selection of date+time always before this date
  const apodLatest = new Date().setHours(23,59,59); 

  //API URL
  const url = "https://api.nasa.gov/planetary/apod?api_key=" + process.env.REACT_APP_NASA_API_KEY + "&date=";

  const fetchPost = async (uDate) => {
    setIsLoading(true);
    setHasError(false);
    try {
      const response = await fetch(
        url + uDate
      );
      if (response.status >= 400) {
        setErrorstate(response.status);
        hasError(true);
      }
      const data = await response.json();
      setApod(data);
    } catch (error) {
      setHasError(true);
    }
    setIsLoading(false);
  };

  function MediaType(media) {
    var mediaLink;
    // Determines whether APOD picture is image or video
    if(media === "image") {
      mediaLink = <img src={apod.url} alt={apod.title}/>;
    } else {
      mediaLink = <iframe title={apod.title} src={apod.url} allowFullScreen> </iframe>;
    }
    return (
      <>
        {mediaLink}
        <p><strong>{apod.title}</strong></p>
        <p>{apod.explanation}</p>
        <p>Copyright: { apod.copyright ? apod.copyright : "NASA (Public Domain)" }</p>
      </>
    );
  }

  function Picture() {
    // Return if server has error
    if (hasError) {
      return (
        <p className="userMsg">Error connecting to server. 
          <br/>
          Server response: {errorState}</p>
      );
    }
    // Return when waiting for API
    else if(isLoading) {
      return <p className="userMsg">Loading...</p>;
    }
    // Return once user has selected date and no longer loading and no error
    else if(userDate) {
      return MediaType(apod.media_type);
    }
    // Default return if date not chosen or no error
    return <p className="userMsg">Select a date between 16-Jun-1995 and today</p>;
  }

  return (
    <>
      <h1>NASA Astronomy Picture of the Day Picker</h1>
      <div className="datepicker">
        <LocalizationProvider dateAdapter={AdapterDateFns} locale={enLocale}>
          <DatePicker 
            minDate={apodEarliest}
            maxDate={apodLatest}
            value={userDate}
            onChange={(newDate) => {
              if (isWithinInterval(
                newDate,
                {start: apodEarliest, end: apodLatest })) {
                setUserdate(newDate);
                fetchPost(format(new Date(newDate), 'yyyy-MM-dd'));
              };
            }}
            renderInput={(params) => <TextField {...params} />}
          />
        </LocalizationProvider>
      </div>
      <Picture />
      <p className="footer"><a href="https://apod.nasa.gov/apod/lib/about_apod.html">About NASA's Astronomy Picture of the Day</a></p>
    </>
  );
}

export default App;
```

### 3d.  Replace App.css file

Erase the contents of the *src/App.css* file and replace with:

```css
body {
  background-color: white;
  color: black;
  display: flex;
  justify-content: center;
}

h1 {
  text-align: center;
}

div.datepicker {
  text-align: center;
  margin-bottom: 1em;
}

p {
  max-width: 75ch;
  margin-right: auto;
  margin-left: auto;
}

iframe {
  width: 100%;
  height: 500px;
}

img {
  width: 100%;
}

p.userMsg {
  text-align: center;
  font-size: x-large;
}

p.footer {
  text-align: center;
  font-size: small;
}
```


### 3e.  Replace App.test.js

When we continuously deploy using GitHub it automatically runs through any test files prior to deploying.  Create-react-app automatically creates ones of these, *src/App.test.js*, for you and this will fail if you make any changes to the *src/App.js* file.  You can either therefore delete *src/App.test.js* or amend it to include updated tests; as tests are good I've done the latter:

```js
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders H1 correctly', () => {
  render(<App />);
  const nasaHeader = screen.getByRole('heading', { name: /nasa astronomy picture of the day picker/i });
  expect(nasaHeader).toBeInTheDocument();
});


test('renders link to APOD about', () => {
  render(<App />);
  const nasaAbout = screen.getByRole('link', { name: /about nasa's astronomy picture of the day/i });
  expect(nasaAbout).toBeInTheDocument();
});

```
## 4.  Add a workflow to Github

We now need to tell GitHub how to deploy your web app to Azure, we do this by adding a workflow.

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
  # change to your application's name in Azure
  AZURE_WEBAPP_NAME: nasa-apod-picker    
  # change both instances of azure-react-apod to the name of your repo
  AZURE_WEBAPP_PACKAGE_PATH: '/home/runner/work/azure-react-apod/azure-react-apod'   
  # set this to the node version to use if not 16
  NODE_VERSION: '16.x'                
  # NASA API key recorded as GitHub secret, delete if you haven't followed APOD section
  REACT_APP_NASA_API_KEY: ${{ secrets.REACT_APP_NASA_API_KEY }} 

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


## 5.  Create an empty Github repo

We are going to enable continuous deployment (CD) on this project via Github.  Therefore create a new repo in Github (without adding README, .gitignore or licence).  We don't actually need to push the project at this stage however.

## 6.  Download publish profile from Azure

Navigate to your Web App within the Azure portal and select the ***Get publish profile*** option, as shown below, to download a file named *nasa-apod-picker.PublishSettings* to your local machine:

[![Azure portal get publish profile](https://www.preciouschicken.com/blog/images/azure-react-apod_get_publish_profile-thumb.png)](https://www.preciouschicken.com/blog/images/azure-react-apod_get_publish_profile.png)

## 7.  Add secrets to GitHub repo

Following the GitHub guide [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) create a new secret with the name ***AZURE_WEBAPP_PUBLISH_PROFILE*** and copy and paste the contents of the publish profile you downloaded earlier.  To make this easier on Linux you can copy a file into the clipboard from the terminal (assuming you have [xsel](http://www.vergenet.net/~conrad/software/xsel/) installed) like so:

```bash
xsel -b < ~/Downloads/nasa-apod-picker.PublishSettings
```

This secret will allow GitHub to access Azure portal in order to deploy your web app on your behalf.

If you have been following the Astronomy Picture of the Day instructions in Section 3, then you will also need to [generate a NASA API key](https://api.nasa.gov/) too.  As before add to GitHub as a repository secret this time with the name ***REACT_APP_NASA_API_KEY*** [^2].

[^2]: If you are going to run this locally by using `npm start` as well as via GitHub then you will also need to create a file named *.env.local* in the root folder of your directory with the content `REACT_APP_NASA_API_KEY=your_key_here`

## 8.  Push
All that is left to do is the initial push of the project to our repo and GitHub and Azure will take care of the rest.  Digital Ocean features a good tutorial on [pushing an existing project to GitHub](https://www.digitalocean.com/community/tutorials/how-to-push-an-existing-project-to-github).

As a reminder the basic terminal commands (once you've created the new empty repo), changing the github PreciousChicken url to your own, are:

```bash
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:PreciousChicken/azure-react-apod.git
git branch -M main
git push -u origin main
```



## Further reading

* [Host a web application with Azure App Service](https://docs.microsoft.com/en-gb/learn/modules/host-a-web-app-with-azure-app-service/?WT.mc_id=azureportalcard_Service_App%20Services_-inproduct-azureportal)
* [https://websitebeaver.com/deploy-create-react-app-to-azure-app-services](https://websitebeaver.com/deploy-create-react-app-to-azure-app-services) - This was a good attempt at getting ninety percent there, but did not work out all the way.
