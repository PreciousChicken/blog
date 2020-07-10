---
title: "Short, sharp Solidity: pure versus view versus .call()"
date: 2020-07-09T16:19:34+01:00
tags: ["ethereum", "truffle", "solidity", ]
categories: ["Blockchain development"]
description: "A worked example of how the read-only Solidity elements pure, view and .call() behave in the Truffle console."
draft: false
---

## Introduction

A quick worked example of how the read-only Solidity elements *pure*, *view* and *.call()* behave in the Truffle console.

At time of writing I'm using: Truffle v5.1.30 (core: 5.1.30), Solidity v0.5.16 (solc-js), Node v14.4.0, Web3.js v1.2.1, and Ubuntu 20.04 LTS (Regolith flavour).

## The smart contract

If you want to follow along then after running `truffle init` in a working folder, copy the following code into *contracts/CallDemo.sol*:

```solidity
// SPDX-License-Identifier: The Unlicencse
pragma solidity ^0.5.16;

contract CallDemo {
  uint private sampleNumber;

  constructor() public {
      sampleNumber = 163;
  }

  // Declared as a view function
  // i.e. will not modify state
  function viewRetrieve() public view returns (uint) {
      return sampleNumber;
  }

  // Declared as a pure function
  // i.e. will not modify or read from state
  function pureRetrieve() public pure returns (uint) {
      return 163;
  }

  // Not declared as either pure or view,
  // will flag as warning in compiler
  function nonViewRetrieve() public returns (uint) {
      return sampleNumber;
  }
  
}
```
and then in *migrations/2_deploy_contract.js* the following:

```javascript
var CallDemo = artifacts.require("CallDemo");

module.exports = function(deployer) {
  // Arguments are: contract
  deployer.deploy(CallDemo);
};
```

## Truffle compile and deploy

Running `truffle deploy` on the above will successfully compile and deploy, but with one warning regarding the last function:

```bash
    /home/call-truffle-demo/contracts/CallDemo.sol:19:3: Warning: Function state mutability can be restricted to view
  function nonViewRetrieve() public returns (uint) {
  ^ (Relevant source part starts here and spans across multiple lines).
```

We'll get to this later.

## Interrogating via truffle console

Dropping into `truffle console` let's retrieve the data from the first function:

```javascript
let app = await CallDemo.deployed()
let viewReturn = await app.viewRetrieve()
viewReturn.words[0]
```

which gives us, correctly, `163`.

So let's run the same on our second function:

```javascript
let pureReturn = await app.pureRetrieve()
pureReturn.words[0]
```

again: `163`.  So by that logic:

```javascript
let nonViewReturn = await app.nonViewRetrieve()
nonViewReturn.words[0]
```

Should give us `163`, right?  Wrong.  We get `Uncaught TypeError: Cannot read property '0' of undefined.`
 
 Alternatively if we just run `nonViewReturn` we get transaction details with no sign of `163`:

 ```solidity
 {
  tx: '0xf383bc9a345686c40dbb52eff8424e72fb921ad44a2aca3f3f20682ec071a05a',
  receipt: {
    transactionHash: '0xf383bc9a345686c40dbb52eff8424e72fb921ad44a2aca3f3f20682ec071a05a',
    transactionIndex: 0,
    blockHash: '0xc840932201f90053a4d17b832a80762122560910b42dbb71eeff78438643a91d',
    blockNumber: 21,
    from: '0x852209101adbaa71516b99197e0cba0f8d102d58',
    to: '0x780a64a641cf32f293b61526abac6859fdc325ad',
    gasUsed: 22077,
    cumulativeGasUsed: 22077,
    contractAddress: null,
    logs: [],
    status: true,
    logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
    rawLogs: []
  },
  logs: []
}
```

So how do we get our number?  We need to add `.call()` to the end of the function:

```javascript
let nonViewReturnCall = await app.nonViewRetrieve.call()
nonViewReturnCall.words[0]
```

And this gives us `163`.

## What's happening?

In essence all three functions are read-only; we are not changing anything on the Ethereum blockchain, we are simply drawing data from it.  The first two functions however are declared respectively as:

- *View*: This declares that no state will be changed.  In other words the function is simply returning state (`sampleNumber`), but not making any changes to the data currently on the blockchain.
- *Pure*: Declares that no state variable will be changed or read.  This is an even more stringent declaration; we are not even reading any data outside of the function itself.  Had we attempted to return the variable `sampleNumber` within a function declare *pure* the contract would have failed to compile.

As the last function was not defined as either of these, and subsequently generated a warning, then the `.call()` function has to be used.  To quote the [Solidity documentation](https://solidity.readthedocs.io/en/latest/types.html#members-of-addresses):

> In order to interface with contracts that do not adhere to the ABI, or to get more direct control over the encoding, the functions `call`, `delegatecall` and `staticcall` are provided.

For a more detailed explanation I would recommend [Calls vs. transactions in Ethereum smart contracts](https://blog.b9lab.com/calls-vs-transactions-in-ethereum-smart-contracts-62d6b17d0bc2).

As an aside you will notice that if you copy and paste *CallDemo.sol* into the online [Remix IDE](https://remix.ethereum.org/) you don't have to worry about the *.call()*  on the last function - the IDE figures out it is needed all by itself. 
