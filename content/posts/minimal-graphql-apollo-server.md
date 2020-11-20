---
title: "Oh-so minimal GraphQL API example with Apollo Server"
date: 2020-11-16T07:48:47Z
tags: ["Node", "GraphQL", "Apollo Server"]
categories: ["Web dev"]
draft: true
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
	    "id": "md",
	    "legs": 6,
	    "binomial": "Musca domestica",
	    "commonName": "housefly",
	    "taxClass": "Insecta",
	    "eats": [ ]
    },
    {
	    "id": "nr",
	    "legs": 8,
	    "binomial": "Neriene radiata",
	    "commonName": "filmy dome spider",
	    "taxClass": "Arachnida",
	    "eats": [ "md" ]
    },
    {
	    "id": "cc",
	    "legs": 2,
	    "binomial": "Corvus corone",
	    "commonName": "carrion crow",
	    "taxClass": "Aves",
	    "eats": [ "nr"]
    },
    {
	    "id": "fc",
	    "legs": 4,
	    "binomial": "Felis catus",
	    "commonName": "cat",
	    "taxClass": "Mammalia",
	    "eats": [ "cc", "nr" ]
    }
  ]
}
```

## The server

Create the file `index.js` and paste the following, which is responsible for the server itself:

```javascript
const { ApolloServer } = require('apollo-server');
const { typeDefs, resolvers  } = require('./schema');

const server = new ApolloServer({ typeDefs, resolvers });

// The `listen` method launches a web server.
server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

## Schemas and resolvers

Two important concepts within GraphQL are:

-  Schema (referred to as `typeDefs` below).  This defines the objects the queries will be calling.  This includes what types make up the object (e.g. integer, string, etc). 
-  Resolvers.  As GraphQL is essentially a layer that you can query sitting on top of a data store (for instance a SQL database), this is the interface where we access the datastore so returning the schema above.  In the case of a SQL database this might be made up of SLQ statements (or a reference to a file that did contain those statements).

## Schema.js

We create the schema and resolvers by pasting the following into a new file named `schema.js`:

```javascript
const { gql } = require('apollo-server');
const db = require('./db.json');

// The statements within quotes are used by GraphQL to provide
// human readable descriptions to developers using the API
const typeDefs = gql`
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
		"a beast's prey"
		eats: [ Beast ]
		"a beast's predators"
		isEatenBy: [ Beast ]
	}

	type Query {
		beasts: [Beast]
		beast(id: ID!): Beast
		calledBy(commonName: String!): [Beast]
	}

	type Mutation {
		createBeast(id: ID!, legs: Int!, binomial: String!, 
			commonName: String!, taxClass: String!, eats: [ ID ]
			): Beast 
	}
`

const resolvers = {

	Query: {

		// Returns array of all beasts.
		beasts: () => db.beasts,

		// Returns one beast given ID.
		// Underscore in the arguments is required and 
		// refers to 'parent', see:
		// https://www.apollographql.com/docs/apollo-server/data/resolvers/#resolver-arguments
		beast: (_, args) => db.beasts.find(element => element.id === args.id),

		// Returns array of beasts where commonName matches 
		// partial string.
		// 'Args' argument is destructured here, 
		// e.g. curly brackets means we don't have to write
		// 'args.commonName'
		calledBy (_, {commonName}) {
			let namedBeasts = [];
			for (let beast of db.beasts) {
				if (beast.commonName.includes(commonName)) {
					namedBeasts.push(beast);
				}
			}
			return namedBeasts;
		}
	},

	Beast: {

		// Returns an array of beasts eaten (e.g. prey).
		// This is a field within db.json, so simply field 
		// for each beast and see what beasts match this id
		// and then return as array.
		// Parent argument is beast that API is currently 
		// focussed on e.g. housefly.
		eats (parent) {
			let prey = [];
			for (let eaten of parent.eats) {
				prey.push(
					db.beasts.find(
						element => element.id === eaten));
			}
			return prey;
		},

		// Returns an array of beasts that eat a given beast 
		// (e.g. predators).
		// This is NOT a field within db.json, so 
		// involves calculating from list of prey within db.json.
		isEatenBy (parent) {
			let predators = [];
			for (let beast of db.beasts) {
				let predatorId;
				for (let eaten of beast.eats) {
					if (eaten === parent.id) {
						predatorId = beast.id;
					}
				}
				if (predatorId) {
					predators.push(
						db.beasts.find(
							element => element.id === predatorId));
				}
			}
			return predators;
		}
	},

	// Adds (and returns) new beast created by user.
	// As this is a minimal example it does not save to 
	// db.json and is in memory only so will delete 
	// on node restart.
	Mutation: {
		createBeast (_, args) {
			let newBeast = {
				id: args.id, 
				legs: args.legs, 
				binomial: args.binomial, 
				commonName: args.commonName,
				taxClass: args.taxClass,
				eats: args.eats 
			}
			db.beasts.push(newBeast);
			return newBeast;
		}
	}
}

exports.typeDefs = typeDefs;
exports.resolvers = resolvers;
```

## Start the server

At the terminal enter:

```bash
node index.js
``` 

Assuming success you should see a message similar to the following:

```bash
Server ready at http://localhost:4000/
```

## Playground

Pointing your browser at the URL given previously you should now see the built-in playground where you can test your API.  

Below the statement *#Write your query or mutation here* enter the following query:

```json
{
  beasts {
    commonName
    legs
  }
}
```

If everything has worked you should see the following response:

[![GraphQL playground](https://www.preciouschicken.com/blog/images/graphql_beast_query.png)](https://www.preciouschicken.com/blog/images/graphql_beast_query.png)

## Arguments

The resolvers we have built also allow us to supply arguments to define our search, so entering:

```json
{
  beast(id: "cc") {
    commonName
    taxClass
  }
}
```


Will pull up data on the crow only:

[![GraphQL playground](https://www.preciouschicken.com/blog/images/graphql_beast_args.png)](https://www.preciouschicken.com/blog/images/graphql_beast_args.png)

Using our API for real we will likely not be hardcoding an argument directly into the query so the *Query Variables* section of the playground allows us to add variables in a similar way as to how we might via a web front page (e.g. a form).  To use this enter in the main query section:

```json
query($input: String!) {
  calledBy(commonName: $input) {
    binomial
    legs
  }
}
```

then in the *Query variables* section below enter the variable as so:

```json
{"input": "housefly"}
```

which should result in the following:

[![GraphQL playground](https://www.preciouschicken.com/blog/images/graphql_beasts_query_variables.png)](https://www.preciouschicken.com/blog/images/graphql_beasts_query_variables.png)

## Mutations

So far we have just been querying the data, we can also amend it via a *mutation*.

Our data set is missing some information; the old woman who has eaten all of these creatures.  Therefore in the playground enter:

```json
mutation {
  createBeast(
    id: "hs"
    legs: 2
    binomial: "Homo sapiens"
    commonName: "old woman"
    taxClass: "Mammalia"
    eats: ["md", "nr", "cc", "fc"]
  ) {
    commonName
    eats {
      commonName
    }
  }
}
```

Success that the data has been entered should appear with a list of the beasts the poor lady has eaten:

[![GraphQL playground](https://www.preciouschicken.com/blog/images/graphql_beasts_query_variables.png)](https://www.preciouschicken.com/blog/images/graphql_beasts_mutation.png)

As this is a minimal example this mutation is in-memory only and isn't saved to disk - if you restart the server the data will be erased.

## Alternative server start

The disadvantage to starting the server with `node index.js` as above, is that every time you make a change to the source code (like add a new resolver or amend the schema), you have to restart from the command line.  Alternatively if you start the server with:

```bash
npx nodemon index.js
```
your code will be monitored and the server will restart automatically.  Very handy when you are making lots of changes.

## Further reading

-  [Introduction to Apollo Server](https://www.apollographql.com/docs/apollo-server/)
-  [Learn to Build a GraphQL Server with Minimal Effort](https://www.freecodecamp.org/news/learn-to-build-a-graphql-server-with-minimal-effort-fc7fcabe8ebd/).  Though I'd argue more effort than this.
