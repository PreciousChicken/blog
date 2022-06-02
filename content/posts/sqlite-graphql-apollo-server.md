---
title: "Sprechen sie GraphSQLite? Or querying data in a SQLite database using GraphQL and Apollo Server"
date: 2022-06-02T12:32:38+01:00
tags: ["Node", "GraphQL", "Apollo Server", "Javascript"]
categories: ["API"]
description: "A worked example of using an Apollo Server to produce a GraphQL API querying a SQLite database."
enableToc: true
draft: false
---

## Introduction

The website [sqlitetutorial.net](https://www.sqlitetutorial.net/) has some good pointers on using the [SQLite](https://www.sqlite.org/) database.  This worked example builds on their [Querying Data in SQLite Database from Node.js Applications](https://www.sqlitetutorial.net/sqlite-nodejs/querying-data-sqlite-database-node-js-applications/) tutorial.  The differences being I'm using the [better-sqlite3](https://www.npmjs.com/package/better-sqlite3) rather than the [sqlite3](https://www.npmjs.com/package/sqlite3) npm package[^1] and I'm using a [GraphQL](https://graphql.org/) API (powered by [Apollo Server](https://www.apollographql.com/docs/apollo-server/)) to present the data (as opposed to `console.log`).  As per the sqlitetutorial.net site I'm using the sample database [chinook.db](https://github.com/lerocha/chinook-database).  This worked example is designed to mimic the tutorial as closely as possible - the queries and responses are almost exactly the same.  It probably isn't a great place to start though if you don't understand the GraphQL fundamentals, in that case you might try [Oh-so minimal GraphQL API example with Apollo Server](https://www.preciouschicken.com/blog/posts/minimal-graphql-apollo-server/) first.

All code can be found at the repo [PreciousChicken/sqlite-graphql-apollo-server](https://github.com/PreciousChicken/sqlite-graphql-apollo-server).

I'm using Manjaro Linux 21.2.6, node v16.13.2, npm v8.3.2, better-sqlite3 v7.5.3 and apollo-server v3.8.1.

## Initialisation

At the terminal create a directory, and install the relevant packages:

```bash
mkdir sqlite-graphql
cd sqlite-graphql
npm init -y
npm install apollo-server better-sqlite3
```

## Download the database

Download and unzip the [sample Chinook.db database](https://www.sqlitetutorial.net/wp-content/uploads/2018/03/chinook.zip) from sqlitetutorial.net into the folder you just created, ensuring it is named ***chinook.db***.  If you have [curl](https://curl.se/) and [7z](https://www.7-zip.org/) installed you could do this at the command line:

```bash
curl https://www.sqlitetutorial.net/wp-content/uploads/2018/03/chinook.zip --output chinook.zip
7z x -tzip chinook.zip chinook.db && rm chinook.zip
```

## Create the server

Copy and paste the following into a new file called ***index.js***:

```javascript
const { ApolloServer } = require('apollo-server');
const { typeDefs, resolvers  } = require('./schema');

const server = new ApolloServer({ typeDefs, resolvers });

// The `listen` method launches a web server.
server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

## Create the schema

Create a file named ***schema.js*** and paste the following:

```javascript
const db = require('better-sqlite3')('chinook.db', { readonly: true, fileMustExist: true});
const { gql } = require('apollo-server');

const typeDefs = gql`
	type Playlist {
		"PlaylistId"
		id: ID
		"Name"
		name: String
	}

	type Customer {
		"Customer ID"
		id: ID
		"First name"
		firstName: String
		"Last name"
		lastName: String
		"Country"
		country: String
		"Email"
		email: String
	}

	type Query {
		playlists: [Playlist]
		playlist(id: ID!): Playlist
		customerLocation(country: String!): [Customer]
	}
`

const resolvers = {
	Query: {
		playlists: () => db.prepare(
			'SELECT DISTINCT Name name ' + 
			'FROM playlists ' + 
			'ORDER BY name'
		).all(),

		playlist: (_, args) => db.prepare(
			'SELECT PlaylistId id, Name name ' +
			'FROM playlists ' + 
			'WHERE PlaylistId  = ?'
		).get(args.id),

		customerLocation: (_, args) => db.prepare(
			'SELECT FirstName firstName, LastName lastName, Email email ' + 
			'FROM customers ' + 
			'WHERE Country = ? ' + 
			'ORDER BY FirstName'
		).all(args.country),
	},
}

exports.typeDefs = typeDefs;
exports.resolvers = resolvers;
```

A close examination will notice that the `SELECT` in each statement renames the elements - so the column `FirstName` in ***chinook.db*** is renamed `firstName`.  This is done so that the data returned by the SQL matches the [GraphQL naming convention](https://www.apollographql.com/docs/apollo-server/schema/schema/#naming-conventions) and which is used for the definitions in the `typeDefs`.  Not strictly necessary, but good practice.

## Start the server

Having written the code, let's start the server.  At the terminal enter:

```bash
node index.js
```

Assuming success you should see a message similar to the following:

```bash
Server ready at http://localhost:4000/
```

## Apollo Sandbox

Pointing our browser at the URL above we should now be invited to ***Query your server***.  Below are listed the examples given in the [sqlitetutorial.net tutorial](https://www.sqlitetutorial.net/sqlite-nodejs/querying-data-sqlite-database-node-js-applications/), broken down into operation (i.e. query), optional variables and expected response.  Give it a go in the Apollo Sandbox [^2].

[^2]: Sandbox has replaced Playground, which has been [retired](https://www.apollographql.com/docs/apollo-server/testing/build-run-queries/#graphql-playground).

### a. Return all distinct playlists

Corresponds with the ***Querying all rows with all() method*** example.

#### Operation

```graphql
query Playlists {
  playlists {
    name
  }
}
```

#### Response

```json
{
  "data": {
    "playlists": [
      {
        "name": "90’s Music"
      },
      {
        "name": "Audiobooks"
      },
      [...]
    ]
  }
}
```

### b. Find a playlist given an ID

Corresponds with the ***Query the first row in the result set*** example.

#### Operation

```graphql
query Playlist($playlistId: ID!) {
  playlist(id: $playlistId) {
    id
    name
  }
}
```

#### Variables

```graphql
{
  "playlistId": 1
}
```

#### Response

```json
{
  "data": {
    "playlist": {
      "id": "1",
      "name": "Music"
    }
  }
}
```
### c. Find all customers in a given country

Corresponds with the ***Query rows with each() method*** example - although we are not using the each() method here, as we are returning the entire set of results with GraphQL and not iterating over the results.

#### Operation

```graphql
query CustomerLocation($country: String!) {
  customerLocation(country: $country) {
    firstName
    lastName
    email
  }
}
```

#### Variables

```graphql
{
  "country": "USA"
}
```

#### Response

```json
{
  "data": {
    "customerLocation": [
      {
        "firstName": "Dan",
        "lastName": "Miller",
        "email": "dmiller@comcast.com"
      },
      {
        "firstName": "Frank",
        "lastName": "Harris",
        "email": "fharris@google.com"
      },
      [...]
    ]
  }
}
```

## Conclusion

Comments, feedback?  Post below.

## Related work

Some other resources I have produced on GraphQL are:

- [Tie down scheme for an Apollo GraphQL server in a Node Docker container](https://www.preciouschicken.com/blog/posts/apollo-server-docker-container/) - Worked example deploying a GraphQL server in a docker container.

- [Oh-so minimal GraphQL API example with Apollo Server](https://preciouschicken.com/blog/posts/minimal-graphql-apollo-server/) - A more comprehensive tutorial on GraphQL APIs.  
- [A no jokes guide to testing a GraphQL API with Jest](https://www.preciouschicken.com/blog/posts/jest-testing-graphql-api/) - Worked example as to testing a GraphQL API.
- [Stacking Vercel, a GraphQL Apollo Server and React](https://www.preciouschicken.com/blog/posts/vercel-apollo-server-react/) - Deploying a GraphQL API on Vercel. 


[^1]: I preferred the documentation of better-sqlite3, plus it claims to be faster.
