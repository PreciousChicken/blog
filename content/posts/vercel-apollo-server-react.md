---
title: "Stacking Vercel, a GraphQL Apollo Server and React"
date: 2021-01-14T17:11:52Z
tags: ["GraphQL", "React", "Vercel", "Apollo Server"]
categories: ["WebDev"]
description: "A worked example how to host a GraphQL API Apollo Server on the Vercel platform with Create-React-App"
enableToc: true
draft: true
---

## Introduction

I enjoy using [Vercel](https://vercel.com/docs/serverless-functions/introduction), it makes it very easy to deploy React single-page-applications.  Given its use of [serverless functions](https://vercel.com/docs/serverless-functions/introduction) however it is not entirely obvious how one might use it to host a GraphQL API Apollo Server.  This worked example for Linux demonstrates how it might be done - borrowing code from my earlier [Oh-so minimal GraphQL API example with Apollo Server](https://www.preciouschicken.com/blog/posts/minimal-graphql-apollo-server/) tutorial.

All code can be found on github at my [vercel-apollo-server-react](https://github.com/PreciousChicken/vercel-apollo-server-react) repo.  The final product is also hosted on Vercel.

## Create-React-App

To get us started:

```bash
npx create-react-app vercel-apollo-server-react
cd vercel-apollo-server-react
npm i apollo-server-micro @apollo/client
```

## GraphQL Server

Vercel looks for a folder named *api* to run serverless functions, so first step is to create it:

```bash
mkdir api
```

Within this folder we need three files: some data for the server to play with, a schema explaining how the data is structured and an initiation of Apollo Server itself.

### The data

As this is a demonstration only, we are going to use a JSON file to act as our datastore (I'm taking inspiration from [There Was an Old Lady Who Swallowed a Fly](https://en.wikipedia.org/wiki/There_Was_an_Old_Lady_Who_Swallowed_a_Fly).  Create the file *api/db.json* and paste the following:

```json
{
  "beasts": [
    {
	    "id": "md",
	    "legs": 6,
	    "binomial": "Musca domestica",
	    "commonName": "housefly"
    },
    {
	    "id": "nr",
	    "legs": 8,
	    "binomial": "Neriene radiata",
	    "commonName": "filmy dome spider"
    },
    {
	    "id": "cc",
	    "legs": 2,
	    "binomial": "Corvus corone",
	    "commonName": "carrion crow"
    },
    {
	    "id": "fc",
	    "legs": 4,
	    "binomial": "Felis catus",
	    "commonName": "cat"
    }
  ]
}
```

### The schema

Create *api/schema.js* and paste the following very basic schema (for a more full-fat schema see my [previous tutorial](https://www.preciouschicken.com/blog/posts/minimal-graphql-apollo-server/)):

```js
import { gql } from 'apollo-server-micro';
import db from './db.json';

// The statements within quotes are used by GraphQL to provide
// human readable descriptions to developers using the API
export const typeDefs = gql`
	type Beast {
		"ID of beast (taken from binomial initial)"
		id: ID
		"number of legs beast has"
		legs: Int
		"a beast's name in Latin"
		binomial: String
		"a beast's name to you and I"
		commonName: String
		"taxonomy grouping"
		taxClass: String
	}

	type Query {
		beasts: [Beast]
	}
`
export const resolvers = {
	Query: {
		// Returns array of all beasts.
		beasts: () => db.beasts,
	}
}
```

### Apollo Server

The serverless function that instantiates Apollo Server itself should be pasted in *api/graphql.js* as: 

```js
import { ApolloServer, makeExecutableSchema } from 'apollo-server-micro'
import { typeDefs, resolvers  } from './schema';

export default new ApolloServer({
    typeDefs,
    resolvers,
    introspection: true,
    playground: true,
}).createHandler({
    path: '/api/graphql',
})
```

The `introspection` and `playground` variables are normally excluded in production, but I've left them in so the GraphQL playground is still accessible.

## Front end

API complete, we now need to edit the React elements within the *src* folder.  Delete the contents of *src/index.js* and replace with:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { createHttpLink, ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';
const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: createHttpLink({ uri: "/api/graphql" }),
});

ReactDOM.render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

Finally delete the contents of *src/App.js*, replacing with:

```jsx
import React from 'react';
import { gql, useQuery } from '@apollo/client';
import './App.css';

function App() {

  const GET_BEASTS = gql`
  query {
    beasts {
      id
      commonName
      legs
      binomial
    }
  }`;

  const { loading, error, data } = useQuery(GET_BEASTS);
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;
  return (
    <div className="App">
      <header className="App-header">
        <h1>Stacking Vercel, a GraphQL Apollo Server and React</h1>
        <p>A table of animals eaten by an old woman:</p>
        <table>
          <thead>
            <tr>
              <th>Name</th>
              <th>Legs</th>
              <th>Binomial</th>
            </tr>
          </thead>
          <tbody>
            {data.beasts.map(beast => 
            <tr key={beast.id}>
              <td>{beast.commonName}</td>
              <td>{beast.legs}</td>
              <td>{beast.binomial}</td>
            </tr>
            )}
          </tbody>
        </table>
        <small>
          <p>This is a demo page to accompany the tutorial <br/>
            <a className="App-link"
              href="https://www.preciouschicken.com/blog/posts/vercel-apollo-server-react">
              preciouschicken.com/blog/posts/vercel-apollo-server-react
            </a></p>
          <p>Consult your own physicican before eating any of the animals on this table.</p>
        </small>
      </header>
    </div>
  );
}

export default App;
```

## Upload to Vercel

There are a number of ways to upload to Vercel, I typically use the Git repo integration, but for the purposes of this walk though we'll use the CLI option.  This of course assumes you have already signed up with Vercel.  Ensuring you are in the root directory login:

```bash
npx vercel login
```

At this point you will have to enter the address used to sign up to Vercel, a confirmation email will be sent and once verified it will confirm in the terminal.  Once done we upload:

```bash
npx vercel --prod
```
Accept all of the default options and once uploaded you'll get the URL Vercel is hosting your app at.  If all has gone well it should look like:

Insert image here



## Version control

This example uses Vercel CLI 21.1.0, node v15.2.0, npm v6.14.11, @apollo/client v3.3.6, apollo-server-micro v2.19.1 and Ubuntu 20.04.1 (Regolith flavour).  If following the instructions does not work first time then this might be the problem.


