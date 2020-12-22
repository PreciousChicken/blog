---
title: "The no jokes guide to testing a GraphQL API with Jest"
date: 2020-12-21T20:11:37Z
tags: ["Jest", "Testing", "JavaScript", "GraphQL"]
categories: ["Web development"]
description: "A worked example using the Jest JavaScript Testing Framework to test a GraphQL API."
draft: false
---

## Introduction

There are many options to test a [GraphQL](https://graphql.org/) API, but in this worked example I'm going to use [Jest](https://jestjs.io/) which is a JavaScript testing framework developed by Facebook.  I've written previously on how to set up a [very minimal GraphQL API](https://www.preciouschicken.com/blog/posts/minimal-graphql-apollo-server/).  This is however a standalone demonstration and relies on a publicly accessible GraphQL API providing [Finnish public transport data](https://digitransit.fi/en/developers/).  As per the title absolutely nothing funny about any of this, just a super concise guide to what you need to do.

All code can be downloaded from my [jest-testing-graphql-api](https://github.com/PreciousChicken/jest-testing-graphql-api) repository.  This worked example requires NodeJS, if you haven't installed it I'd recommend doing so following this [StackOverflow answer](https://stackoverflow.com/a/24404451).

I'm running Ubuntu 20.04.1 (Regolith flavour), Node 15.2.0, npm 6.14.9 and Jest 26.6.3.

## Initialisation

At the terminal create a directory, and install the relevant node packages:

```bash
mkdir jest-testing-graphql-api
cd jest-testing-graphql-api
npm init -y
npm install --save-dev isomorphic-fetch jest 
```

The `--save-dev` option tells npm that we will just be using these packages in development and not production.  Although that is certainly the case with Jest, as it is only used for testing, it might not be the case with isomorphic-fetch; so depending on your project you might not want to install this package with the `--save-dev` option.

## The test condition

Create a file named `ark.test.js` and copy and paste the following:

```javascript
require('isomorphic-fetch');

// Test name as defined by developer
test('the stop is Arkadian puisto', () => {
	
	// The result we are expecting from the GraphQL API
	const arkP = {
		"stop": {
			"name": "Arkadian puisto", 
			"lat": 60.17112, 
			"lon": 24.93338
		}
	};

	// The URL of the GraphQL API server
	return fetch('https://api.digitransit.fi/routing/v1/routers/hsl/index/graphql', {
		method: 'POST',
		headers: { 'Content-Type': 'application/json' },
		// The query we are sending to the GraphQL API
		body: JSON.stringify({ query: 
			`query {
				stop(id: "HSL:1040129") {
					name
					lat
					lon
				}
			}` 
		}),
	})
	.then(res => res.json())
	// The test condition itself
	.then(res => expect(res.data).toStrictEqual(arkP));
});
```

This file contains the test itself.  The `test` function tells Jest what to expect, queries the GraphQL API for a response and then checks that this matches our expectation.  Unsurprisingly the `test` in the file name informs Jest that it needs to process this file; alternatively the file can be placed in a directory named `__tests__`.

As `fetch` is using a Promise to return the data we use a `return` followed by a `.then()` to handle the asynchronous nature of the test as recommended in the [Jest documentation](https://jestjs.io/docs/en/asynchronous#promises).

## Insert a test script into the package

Next alter our `package.json` file to include the ability to run a test from the command line by replacing the `scripts` object so that the file reads:

```json
{
	"name": "jest-testing-graphql-api",
	"version": "1.0.0",
	"description": "",
	"main": "index.js",
	"scripts": {
		"test": "jest"
	},
	"keywords": [],
	"author": "",
	"license": "ISC",
	"devDependencies": {
		"isomorphic-fetch": "^3.0.0",
		"jest": "^26.6.3"
	}
}
```

## Run the test

From the terminal run `npm test`.  If the test ran successfully then you should see the following output:

[![Jest command line output](https://www.preciouschicken.com/blog/images/jest-test.png)](https://www.preciouschicken.com/blog/images/jest-test.png)

To see an error message change a character in the `arkP` variable and run again.

## Conclusion and Further reading

If you have found this useful or have feedback, please do leave a comment below.  Some resources I found useful when writing this were:

- [Jest: Getting Started](https://jestjs.io/docs/en/getting-started).  Introduction to Jest.
- [4 Simple Ways to Call a GraphQL API](https://www.apollographql.com/blog/4-simple-ways-to-call-a-graphql-api-a6807bcdb355/).  Nice article on the various ways you can interface with a GraphQL API.
