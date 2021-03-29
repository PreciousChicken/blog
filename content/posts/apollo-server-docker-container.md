---
title: "Tie down scheme for an Apollo GraphQL server in a Node Docker container"
date: 2021-03-27T21:08:34Z
tags: ["Docker", "Node", "Apollo-server", "GraphQL", "Javascript"]
categories: ["SysAdmin"]
description: "Worked example of deploying an Apollo GraphQL server in a Node Docker Container"
enableToc: true
draft: false
---

## Introduction

[GraphQL](https://graphql.org/) is a query language for APIs, while [Apollo Server](https://www.apollographql.com/) is a popular server used for providing GraphQL APIs.  This post is a concise handrail as to how to put an Apollo GraphQL server in a Node.js [Docker container](https://www.docker.com/why-docker) - a "standardized unit of software that allows developers to isolate their app from its environment."  This tutorial uses the Linux command line and assumes you have already [installed Docker](https://docs.docker.com/get-docker/) and [Node](https://nodejs.org/en/download/package-manager/).

All code can be found in the [PreciousChicken/apollo-server-docker-container](https://github.com/PreciousChicken/apollo-server-docker-container) repository.


## Initialisation

At the terminal create a directory, and install the relevant packages:

```shell
mkdir apollo-docker
cd apollo-docker
npm init -y
npm install apollo-server graphql
```

## Create the GraphQL server, schema et al

Not good practice to put everything into the same file, but as this is a demo only create a file named *index.js* and copy / paste the below:

```javascript
const { ApolloServer, gql } = require('apollo-server');

const data = {
  "beasts": [
    {
	    "id": "md",
	    "legs": 6,
	    "binomial": "Musca domestica",
	    "commonName": "housefly",
    },
    {
	    "id": "nr",
	    "legs": 8,
	    "binomial": "Neriene radiata",
	    "commonName": "filmy dome spider",
    },
    {
	    "id": "cc",
	    "legs": 2,
	    "binomial": "Corvus corone",
	    "commonName": "carrion crow",
    },
    {
	    "id": "fc",
	    "legs": 4,
	    "binomial": "Felis catus",
	    "commonName": "cat",
    }
  ]
};

const typeDefs = gql`
	type Beast {
		id: ID
		legs: Int
		binomial: String
		commonName: String
	}

	type Query {
		beasts: [Beast]
	}
`;

const resolvers = {
	Query: {
		// Returns array of all beasts.
		beasts: () => data.beasts
	}
};

const server = new ApolloServer({ typeDefs, resolvers });

// The `listen` method launches a web server.
server.listen(4000);
```

This creates a very basic Apollo server with a minimal GraphQL schema.  Running `node index.js` now would spin this up and allow interaction with the server - but the aim is to run within a Docker container, not directly on our machine: which is the next step.

## Dockerfile

A dockerfile is a set of instructions that builds our image.  Therefore create a file named *Dockerfile* and copy / paste the following:

```dockerfile
# Uses the node base image with the latest LTS version
FROM node:14.16.0
# Informs Docker that the container listens on the 
# specified network ports at runtime
EXPOSE 4000
# Copies index.js and the two package files from the local 
# directory to a new app directory on the container
COPY index.js package.json package-lock.json  app/
# Changes working directory to the new directory just created
WORKDIR /app
# Installs npm dependencies on container
RUN npm ci
# Command container will actually run when called
CMD ["node", "index.js"]
```

A keen reading might spot that `npm ci` is used in preference to `npm install`, this command is meant to be used in automated environments as explained by [Demystifying npm install, npm ci & package-lock.json](https://medium.com/@Cyclodex/demystifying-npm-install-npm-ci-package-lock-json-2807fc0ee404).


## Build the image

Next we tell Docker to use the newly created Dockerfile to create the image:

```bash
docker build -t preciouschicken-apollo .
```

Docker will pull down the parent image and run through the Dockerfile.

The *-t* option names the image created, in this case as *preciouschicken-apollo*.  To confirm success all images created can be listed with:

```bash
docker image ls
```

## Run the container

Now the image has been created, next run an instance of it:

```bash
docker run -p 4000:4000 --name apollo-server -d preciouschicken-apollo
```

The options specified here are:

-  *-p 4000:4000* - Connects the port on the local machine to the port on the container.
-  *--name apollo-server* - Names the actual running instance of the image, useful for identification.
-  *-d* - Runs the instance detached: that is hands your terminal back so you can type other commands.

To check the container is running it can be listed with:

```bash
docker ps
```

## Interact with the GraphQL server

If all went well Apollo server should be running in the container and listening on port 4000.    Pointing your browser at [localhost:4000](http://localhost:4000/) should display the built-in playground where you can test your API.  

Below the statement *#Write your query or mutation here* enter the following query:

```json
{
  beasts {
    commonName
    legs
  }
}
```

This should produce a response similar to the below:

[![GraphQL playground](https://www.preciouschicken.com/blog/images/graphql_beast_query-thumb.png)](https://www.preciouschicken.com/blog/images/graphql_beast_query.png)

## Stop the container

When we are done testing we can halt the container with:

```bash
docker stop apollo-server
```

## Conclusion

Comments, feedback?  Post below.

Oh and if you are wondering, what is a tie down scheme anyway?  It's a schematic for how to fasten a load within an aircraft or ISO container.  Think of it like a very weak joke.

## Related work

[Node.js Docker Best Practices](https://github.com/goldbergyoni/nodebestpractices#8-docker-best-practices) provides tips on production ready Docker containers.

Some other resources I have produced on GraphQL are:

- [Oh-so minimal GraphQL API example with Apollo Server](https://preciouschicken.com/blog/posts/minimal-graphql-apollo-server/) - A more comprehensive tutorial on GraphQL APIs.  
- [A no jokes guide to testing a GraphQL API with Jest](https://www.preciouschicken.com/blog/posts/jest-testing-graphql-api/) - Worked example as to testing a GraphQL API.
- [Stacking Vercel, a GraphQL Apollo Server and React](https://www.preciouschicken.com/blog/posts/vercel-apollo-server-react/) - Deploying a GraphQL API on Vercel. 

## Version Control

This was written on a Linux machine running Manjaro 21.0 Omara, using Docker Server 20.10.5, npm 7.7.5 and node 14.16.0.

