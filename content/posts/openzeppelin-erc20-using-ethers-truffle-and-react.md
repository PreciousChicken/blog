---
title: "OpenZeppelin's ERC20 using Ethers, Truffle and React"
date: 2020-07-01T17:08:04+01:00
tags: ["ethereum", "truffle", "ethers.js", "react", "metamask"]
categories: ["Blockchain development"]
description: "A demonstration of OpenZeppelin's ERC20 using Ethers, Truffle and React"
draft: false
---

## Introduction

A step-by-step demonstration of messing around with ERC20 Tokens in React.  This should definitely not be taken as best practice on how to use ERC20.  I wrote this for familiarisation purposes, not production.    

Due to the variety of moving parts there's quite a lot of configuration control that needs to go up front.  Your mileage may vary if you are using different versions - in fact expect it not to work at all, there appear to be lots of breaking changes in the Ethereum world.  So components used: node v14.4.0, truffle v5.1.30, ganache v.2.4.0, metamask v7.7.9, @openzeppelin/contracts@3.1.0, ethers@5.0.3, create-react-app v3.4.1 and my OS is Ubuntu 20.04 LTS.

## Prerequisites

-  [NodeJS](https://nodejs.org) - If you haven't installed it before, I found installing it using the Node Version Manager as suggested on this [Stack Overflow answer](https://stackoverflow.com/a/24404451/6333825) to cause less aggravation than downloading via the official website.
-  If you've previously installed `create-react-app` globally via `npm install -g create-react-app`, then uninstall it with the command `npm uninstall -g create-react-app` so you are using the latest version in the step below. 

## Ganache

[Ganache](https://www.trufflesuite.com/ganache), part of the Truffle suite, positions itself as a "one-click blockchain."  It allows a developer to host an Ethereum blockchain quickly on a local machine and provides a number of accounts (or wallets) already full of pretend Ether.  This is much easier and quicker than deploying on either a testnet (e.g. Ropsten) or the main Ethereum network (which costs actual money).

Download the Appimage from their [homepage](https://www.trufflesuite.com/ganache) and run (I've blogged previously on how [I like to install Appimages](https://www.preciouschicken.com/blog/posts/where-do-i-put-appimages/), but clearly install how you want).  Once the application is running select the _Quickstart Ethereum_ option when you are invited to 'Create a workspace.'  You should see be presented with the following screen:

[![Ganache](https://www.preciouschicken.com/blog/images/ganache.png)](https://www.preciouschicken.com/blog/images/ganache.png)

which gives a list of accounts (or wallets) Ganache has created for you and the pretend Ether they contain.

There are other non-GUI ways of running a local blockchain, and although I'm generally a fan of the terminal, this is such a neat solution I'm going to use it.

## Truffle

Although Ganache is part of the Truffle suite; the main course, if you like is [Truffle](https://www.trufflesuite.com/truffle) itself.  Truffle is a development framework for Ethereum which allows you to deploy and test smart contracts quickly.

Create a directory that will hold our project:

```bash
mkdir erc20-pct
cd erc20-pct
```

Now we initialise Truffle which will install a number of [default files and folders](https://www.trufflesuite.com/tutorials/getting-started-with-drizzle-and-react#directory-structure):

```bash
truffle init
```

## OpenZeppelin ERC20

ERC20 is a standard for tokens that applies on the Ethereum network (ERC standing for Ethereum Request for Comments) that, ensures interoperability of these assets across the network.

Having a standard for tokens is a big deal as tokens were a central feature of Ethereum as laid out in the original [whitepaper](https://ethereum.org/en/whitepaper/#token-systems):

> On-blockchain token systems have many applications ranging from sub-currencies representing assets such as USD or gold to company stocks, individual tokens representing smart property, secure unforgeable coupons, and even token systems with no ties to conventional value at all, used as point systems for incentivization. Token systems are surprisingly easy to implement in Ethereum. The key point to understand is that a currency, or token system, fundamentally is a database with one operation: subtract X units from A and give X units to B, with the provision that (1) A had at least X units before the transaction and (2) the transaction is approved by A. 

Using this standard therefore ensures that everyone is using a common list of rules so allowing a token developed by one person to be traded across the system.  There are a number of [other tokens](https://crushcrypto.com/ethereum-erc-token-standards/), however I'm using ERC20 primarily as it is the most well used.

Although it is possible to roll your own ERC20 by implementing the interface provided in the standard, it makes sense to use one that has been created and thoroughly tested by a specialised third party, in this case [OpenZeppelin](https://openzeppelin.com). 

To make OpenZeppelin's ERC20 available to  we therefore install it with npm (initialising this first):

```bash
npm init -y
npm install @openzeppelin/contracts
```

## Smart Contract: PreciousChickenToken

The ERC20 we will be creating for this demo will be called the PreciousChickenToken (Symbol: PCT), and to do so we need to create a smart contract.

Using your favourite text editor (mine happens to be vim) create and edit the following file:

```bash
vim contracts/PreciousChickenToken.sol
```

Copy and paste the following text (or alternatively download it from my [github repository](https://github.com/PreciousChicken/openzeppelin-erc20-using-ethers-truffle-and-react)):

```sol
// SPDX-License-Identifier: The Unlicencse
pragma solidity ^0.6.1;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract PreciousChickenToken is ERC20 {

    // In reality these events are not needed as the same information is included
    // in the default ERC20 Transfer event, but they serve as demonstrators
    event PCTBuyEvent (
        address from,
        address to,
        uint256 amount
    );
    
    event PCTSellEvent (
        address from,
        address to,
        uint256 amount
    );

    address private owner;
    mapping (address => uint256) pendingWithdrawals;
   
    // Initialises smart contract with supply of tokens going to the address that
    // deployed the contract
    constructor(uint256 _initialSupply) public ERC20("PreciousChickenToken", "PCT") {
        _mint(msg.sender, _initialSupply);
        owner = msg.sender;
    }

    // A wallet sends Eth and receives PCT in return
    function buyToken(uint256 _amount) external payable {
        // Ensures that correct amount of Eth sent for PCT
        // 1 ETH is set equal to 1 PCT
        require(_amount == ((msg.value / 1 ether)), "Incorrect amount of Eth.");
        transferFrom(owner, msg.sender, _amount);
        emit PCTBuyEvent(owner, msg.sender, _amount);
    }
    
    // A wallet sends PCT and receives Eth in return
    function sellToken(uint256 _amount) public {
        pendingWithdrawals[msg.sender] = _amount;
        transfer(owner, _amount);
        withdrawEth();
        emit PCTSellEvent(msg.sender, owner, _amount);
    }

    // Using the Withdraw Pattern to remove Eth from contract account when user
    // wants to return PCT
    // https://solidity.readthedocs.io/en/latest/common-patterns.html#withdrawal-from-contracts
     function withdrawEth() public {
        uint256 amount = pendingWithdrawals[msg.sender];
        // Pending refund zerod before to prevent re-entrancy attacks
        pendingWithdrawals[msg.sender] = 0;
        msg.sender.transfer(amount * 1 ether);
    }
}
```
This contract when called generates a number of 'PreciousChickenTokens' and allocates them to the address that deployed the contract - in our case that will be the first address listed in Ganache (marked as 'Index 0').  It then has a function to allow users to buy PCTs in exchange for Ether (with the conversion rate set at 1 Eth equal to one PCT), and to sell those PCT back in return for Ether.  Any Ether collected goes to the contract address itself.

Again it is worth re-iterating this code is intended as a demonstrator only and is not intended to be a model for anything production ready that is exposed to financial risk.

Next is the code that deploys the contract we've just written.  Create the following file:

```bash
vim migrations/2_deploy_contract.js
```

Copy and paste the following into the file you've just created:

```javascript
var PreciousChickenToken = artifacts.require("PreciousChickenToken");

module.exports = function(deployer) {
  // Arguments are: contract, initialSupply
  deployer.deploy(PreciousChickenToken, 1000);
};
```

This code is responsible for telling Truffle to deploy the smart contract listed, which we've just written.  It also serves to provide arguments to the constructor method within _PreciousChickenToken.sol_, this had one argument called _\_initialSupply_ which sets the amount of tokens created on deployment of the contract (i.e. one thousand).

