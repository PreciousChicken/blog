---
enableToc: true
title: "Avoiding Call Revert Exception Error when accessing Truffle Ganache via Ethers in Node"
date: 2020-07-20T09:30:29+01:00
tags: ["ethereum", "truffle", "ganache", "solidity", "node", "ethers.js"]
categories: ["Blockchain development"]
description: "Example code showing how to connect Truffle Ganache via Ethers in Node."
draft: false
---

## Introduction

My default way of connecting to a local instance of the Ethereum blockchain using Truffle Ganache is via the browser using Metamask.  Using ethers to connect via node.js is however a little different.  As the [Ethers documentation](https://docs.ethers.io/v5/), at time of writing, contains few specific mentions of Truffle (primarily I suspect as the lead developer [doesn't use it](https://github.com/ethers-io/ethers.js/issues/71#issuecomment-441057904)), I got a couple of `call revert exception` errors before I figured out what I was doing wrong.

## Setup

Just in case you are completely new to this, some initial set up steps at the terminal:

```bash
mkdir justsayhi
cd justsayhi
truffle init
npm init -y
npm install ethers
```

I am assuming you have already installed Node, Truffle and Ganache, if not see my earlier post [PreciousChickenToken: A guided example of OpenZeppelin's ERC20 using Ethers, Truffle and React](https://www.preciouschicken.com/blog/posts/openzeppelin-erc20-using-ethers-truffle-and-react/).

Now would also be a good time to fire up Ganache, selecting the *Quickstart Ethereum* option when offered.

## The JavaScript

Create a file *node_server.js* and copy and paste:

```javascript
var ethers = require('ethers');
var JustSayHi = require('./build/contracts/JustSayHi.json');

// This is the localhost port Ganache operates on
const url = "http://127.0.0.1:7545";
const provider = ethers.providers.getDefaultProvider(url);

const contractAddress ='0x02e68a4a2B539451F7d02b166B3376DBc7473F75';

// Connect to the network
// We connect to the Contract using a Provider, so we will only
// have read-only access to the Contract
let contract = new ethers.Contract(contractAddress, JustSayHi.abi, provider);

try {
	contract.sayHi().then(msg => console.log(msg));
} catch (e) {
	console.log(e);
}
```

## The SmartContract

Create a file *contracts/JustSayHi.sol* and copy and paste:

```solidity
// SPDX-License-Identifier: Unlicencse
pragma solidity ^0.5.1;

contract JustSayHi {
    function sayHi() public pure returns (string memory) {
        return "Hi";
    }
}
```

Create a file *migrations/2_deploy_contract.js* and copy and paste:

```javascript
var JustSayHi = artifacts.require("JustSayHi");

module.exports = function(deployer) {
  // Arguments are: contract
  deployer.deploy(JustSayHi);
};
```

## Deploy the contract

From the terminal run:

```bash
truffle deploy
```

If all good we should get a screen that looks like:

[![Truffle deploying JustSayHi](https://www.preciouschicken.com/blog/images/just_say_hi_truffle_deploy.png)](https://www.preciouschicken.com/blog/images/just_say_hi_truffle_deploy.png)

Copy the highlighted *contract address* above and paste it into the *contractAddress* variable line in *node_server.js*, which is this line in my example above:

```javascript
const contractAddress ='0x02e68a4a2B539451F7d02b166B3376DBc7473F75';
```

## Run using Node.js

From the terminal:

```bash
node node_server.js
```

And if all is well you should see a nice, friendly `Hi` in response.

## Configuration

At time of writing I'm using: Truffle v5.1.34 (core: 5.1.34), Solidity v0.5.16 (solc-js),  Node v14.4.0, Web3.js v1.2.1, ethers v5.07 and Ubuntu 20.04 LTS.



