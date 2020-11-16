---
title: "Oh-so minimal GraphQL API example with Apollo Server"
date: 2020-11-16T07:48:47Z
tags: ["Node", "GraphQL", "Apollo Server"]
categories: ["Web dev"]
draft: false
---

## Introduction

[GraphQL](https://graphql.org/) is a query language for APIs designed by Facebook and intended as a replacement for REST.  [Apollo Server](https://www.apollographql.com/docs/apollo-server/) is a community maintained open-source server designed for GraphQL that runs on node.  This post is a really minimal example of getting Apollo Server running - just so one gets a feel for the core parts.  I'm therefore using a simple json flat file as a data store, and querying that with JavaScript; this isn't particularly realistic as to how you would use GraphQL for real, but as it is vanilla as you can get it doesn't distract.

All code can be downloaded from my [minimal-graphql-apollo-server](https://github.com/PreciousChicken/minimal-graphql-apollo-server) repository.

I'm running Ubuntu 20.04.1 (Regolith flavour), Node 15.2.0, GraphQL 15.4.0 and Apollo Server 2.19.0.

## Initialisation

At the terminal create a directory, and install the relevant software:

```shell
mkdir minimal-graphql-apollo-server
cd minimal-graphql-apollo-server
npm init -y
npm install apollo-server graphql
```


## The data

We need some data to query so create a file named `db.json` and insert the following text (clearly inspired by [There was an old lady who swallowed a fly](https://en.wikipedia.org/wiki/There_Was_an_Old_Lady_Who_Swallowed_a_Fly)):

```json
{
  "beasts": [
    {
	    "id": 1,
	    "legs": 6,
	    "binomial": "Musca domestica",
	    "commonName": "housefly",
	    "class": "Insecta",
	    "eats": [],
	    "isEatenBy": [2]
    },
    {
	    "id": 2,
	    "legs": 8,
	    "binomial": "Neriene radiata",
	    "commonName": "filmy dome spider",
	    "class": "Arachnida",
	    "eats": [1],
	    "isEatenBy": [3]
    },
    {
	    "id": 3,
	    "legs": 2,
	    "binomial": "Corvus corone",
	    "commonName": "carrion crow",
	    "class": "Aves",
	    "eats": [2],
	    "isEatenBy": [4]
    },
    {
	    "id": 4,
	    "legs": 4,
	    "binomial": "Felis catus",
	    "commonName": "cat",
	    "class": "Mammalia",
	    "eats": [3],
	    "isEatenBy": []
    }
  ]
}
```

## The server

Create the file `index.js` and paste the following, which is responsible for the server itself:

```javascript
const { ApolloServer } = require('apollo-server');
const { resolvers, typeDefs } = require('./schema');

const server = new ApolloServer({ typeDefs, resolvers });

// launches web server
server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

## Schemas and resolvers

GraphQL relies upon:

-  Schemas.  This defines the objects the queries will be calling and the types (e.g. integer, string, etc) one can expect. 
-  Resolvers.



## Further reading

-  [Introduction to Apollo Server](https://www.apollographql.com/docs/apollo-server/)
-  [Learn to Build a GraphQL Server with Minimal Effort](https://www.freecodecamp.org/news/learn-to-build-a-graphql-server-with-minimal-effort-fc7fcabe8ebd/).  Though I'd argue more effort than this.
