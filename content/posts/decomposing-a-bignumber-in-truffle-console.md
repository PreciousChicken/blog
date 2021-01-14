---
enableToc: true
title: "Decomposing a BigNumber in Truffle Console"
date: 2020-06-18T14:15:17+01:00
tags: ["ethereum", "truffle"]
categories: ["Blockchain development"]
description: "How to convert a BigNumber returned by the Truffle Console into individual values"
draft: false
---

## The BigNumber problem

The [Truffle Console](https://www.trufflesuite.com/docs/truffle/getting-started/using-truffle-develop-and-the-console) when queried tends to return BigNumbers, which look a bit like this:

```bash
BN {
  negative: 0,
  words: [ 10000, <1 empty item> ],
  length: 1,
  red: null
}
```

Converting this into individual values, i.e. integers, on the console is not obvious; so an aide memoire follows.  I'm using Truffle v5.1.30, node v14.4.0 and npm package @openzeppelin/contracts@3.0.2; my OS of choice is Ubuntu 20.04 LTS.

## The smart contract

Just so we are on the same page, here is the very simple smart contract I'm using as an example which is saved at _contracts/PreciousChickenCoin.sol_:

```solidity
pragma solidity ^0.6.2;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract PreciousChickenCoin is ERC20 {
  uint public initialSupply = 10000;
  constructor() public ERC20("PreciousChickenCoin", "PCC") {
      _mint(msg.sender, initialSupply);
  }
}
```

and deployed as _migrations/2&#95;deploy&#95;contract.js_:

```javascript
var PreciousChickenCoin = artifacts.require("PreciousChickenCoin");

module.exports = function(deployer) {
  deployer.deploy(PreciousChickenCoin);
};
```

The ERC20 import is achieved using:

```bash
npm install @openzeppelin/contracts
```

Additionally with the version of Truffle I'm using, v5.1.30, you have to explicitly tell it to use a version of Solidity that works with OpenZeppelin 3.0.2.  You do that by modifying the following lines of the _truffle-config.js_ file:

```javascript
  compilers: {
    solc: {
      version: "^0.6.0",    // Fetch exact version from solc-bin (default: truffle's version)
    }
  }
```

## Aim

The objective here is to use OpenZeppelin's ERC20 [balanceOf()](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-balanceOf-address-) method to find out how many PreciousChicken coins the `msg.sender` has received on instantiating the smart contract.


This isn't a Ethereum tutorial, so I'm going to assume that you have compiled, migrated, are running something like [Ganache](https://www.trufflesuite.com/ganache), etc, etc and have started the truffle console with `truffle console` at the command line.

All the commands below will be at the _truffle(ganache)>_ prompt.

## Assign the deployed smart contract to a variable with a Promise

First we need to assign the deployed instance to a variable using the JavaScript Promise API (i.e. `.then()`).

```solidity
PreciousChickenCoin.deployed().then(function(instance) {app = instance})
```

which should return `undefined`.

## Discover the `msg.sender` address

Next find the `msg.sender` address, which we can do at the terminal:

```solidity
web3.eth.getAccounts()
```

this should return all the accounts present in Ganache:

```solidity
[
  '0x48a8163fA169Fd9eA3C14409aF29FF22699cE53f',
  '0x524b29d0817412fFc470a50Efc1241B3999Dfce9',
  '0x91167E8DaDa250B4465Bf0D63b679E27DBe69De3',
  '0xE98135dB5a4CB6b6F65C3AFC9fa3868e0845BE68',
  '0x7137C7a475918040d0eDC2b35cCF4911a3B590A6',
  '0xe50fFEeCA1beE4D0a83dF11a97CFbf78b6eB8aCc',
  '0x2E11471f2290C248fDC9ee338B9e2a8Cc1EC1773',
  '0xC90A29e53F07A6C6faC7A412bdF39816478fA6F4',
  '0xb0e5E47c8936640C0479387a82545A05051Bf331',
  '0x4224E151127da145D76dFDc01Aa1b9023EF4ed34'
]
```

Copy the first address (assuming it has been used - you can check this using Ganache).

## Use the ERC20 balanceOf() method

Using the address you copied above:

```solidity
app.balanceOf("0x48a8163fA169Fd9eA3C14409aF29FF22699cE53f")
```

should return the BigNumber (BN) we are interested in:

```solidity
BN {
  negative: 0,
  words: [ 10000, <1 empty item> ],
  length: 1,
  red: null
}
```

Now at this point you might think:

```solidity
app.balanceOf("0x48a8163fA169Fd9eA3C14409aF29FF22699cE53f").words[0]
```

would provide you with the balance, but in actual fact we get a string of errors.

## Assign balance to a variable with a Promise

Due to the asynchronous nature of JavaScript we have to assign the balance first:

```solidity
app.balanceOf("0x48a8163fA169Fd9eA3C14409aF29FF22699cE53f").then(function(balance) { balanceInstance = balance})
```

which returns `undefined`.


## Finally get the variable

Now:

```solidity
balanceInstance.words[0]
```

returns the value we want: `10000`.  

If you found this useful, or have feedback, please comment below.

## References

The [How to Build Ethereum Dapp (Decentralized Application Development Tutorial)](https://www.youtube.com/watch?v=3681ZYbDSSk) by [DApp University](https://www.dappuniversity.com) was particularly useful, especially the [console demonstration](https://www.youtube.com/watch?v=3681ZYbDSSk&feature=youtu.be&t=21m50s) section.
