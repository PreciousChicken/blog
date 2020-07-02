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

-  [NodeJS](https://nodejs.org) - If you haven't installed it before, I found installing it using the Node Version Manager (nvm) as suggested on this [Stack Overflow answer](https://stackoverflow.com/a/24404451/6333825) to cause less aggravation than downloading via the official website.
-  If you've previously installed `create-react-app` globally via `npm install -g create-react-app`, then uninstall it with the command `npm uninstall -g create-react-app` so you are using the latest version in the step below. 

## Ganache

[Ganache](https://www.trufflesuite.com/ganache), part of the Truffle suite, positions itself as a "one-click blockchain."  It allows a developer to host an Ethereum blockchain quickly on a local machine and provides a number of accounts (or wallets) already full of pretend Ether.  This is much easier and quicker than deploying on either a testnet (e.g. Ropsten) or the main Ethereum network (which costs actual money).

Download the Appimage from their [homepage](https://www.trufflesuite.com/ganache) and run (I've blogged previously on how [I like to install Appimages](https://www.preciouschicken.com/blog/posts/where-do-i-put-appimages/), but clearly install how you want).  Once the application is running select the _Quickstart Ethereum_ option when you are invited to 'Create a workspace.'  You should be presented with something similar to the following:

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

Now we (globally) install and then initialise Truffle which will install a number of [default files and folders](https://www.trufflesuite.com/tutorials/getting-started-with-drizzle-and-react#directory-structure):

```bash
npm install -g truffle
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

This code is responsible for telling Truffle to deploy the smart contract listed, which we've just written.  It also serves to provide arguments to the constructor method within _PreciousChickenToken.sol_, this had one argument called *\_initialSupply* which sets the amount of tokens created on deployment of the contract (i.e. one thousand).

Lastly we need to make some amends to our existing truffle configuration file:

```bash
vim truffle-config.js
```

Delete the entire contents and replace with this:

```javascript
const path = require("path");

module.exports = {
	contracts_build_directory: path.join(__dirname, "client/src/contracts"),
	networks: {
	},
	mocha: {
	},
	compilers: {
		solc: {
			version: "^0.6.0",    // Fetch exact version from solc-bin (default: truffle's version)
		}
	}
}
```

So a number of changes have been made to the original:

- Comments have been deleted.  Clearly normally we'd keep them in, but for the purposes of this guide it is easier to see what's happening if we delete them.
- A build directory has been set in a sub-directory.  We are going to be putting our React front end in a subdirectory called *client*.  As the front end needs to have access to the smart contract and can't view the root directory, we need to tell truffle to build the files in a directory it does have access to i.e. *client/src/contracts*.
- We've changed the solidity compiler to minimum version 0.6.0.  This is different to the Truffle default and without doing so the latest version of the OpenZeppelin ERC20 (3.1.0) will not compile.

If we weren't using React or the latest version of OpenZeppelin's contracts therefore, we could keep this as is.


## Build the front end

As we are viewing this in React we now need to generate the front end, to do this we are going to use create-react-app to create a new sub-folder:

```bash
npx create-react-app client
```

Once finished you will have your standard create-react-app files and folders, but there are a couple of additional installations we need to do:

```bash 
cd client
npm install ethers react-bootstrap bootstrap
```
The first of these packages, [ethers.js](https://docs.ethers.io/v5/), is the most important - it aiming to be a "complete and compact library for interacting with the Ethereum Blockchain and its ecosystem"; the second two are for the purposes of UI.

Enter the following file with your text editor:

```bash
vim src/App.js
```

Delete all the content, and replace with:

```javascript
import React, { useState } from 'react';
import './App.css';
import { ethers } from "ethers";
import PreciousChickenToken from "./contracts/PreciousChickenToken.json";
import { Button, Alert } from 'react-bootstrap';
import 'bootstrap/dist/css/bootstrap.min.css';

// Needs to change to reflect current PreciousChickenToken address
const contractAddress ='0x3f45f12c12a6CCA5A0F0aA48Ec2214165FC6E7D0';

let provider;
let signer;
let erc20;
let noProviderAbort = true;

// Ensures metamask or similar installed
if (typeof window.ethereum !== 'undefined' || (typeof window.web3 !== 'undefined')) {
	try{
		// Ethers.js set up, gets data from MetaMask and blockchain
		provider = new ethers.providers.Web3Provider(window.ethereum);
		signer = provider.getSigner();
		erc20 = new ethers.Contract(contractAddress, PreciousChickenToken.abi, signer);
		noProviderAbort = false;

	} catch(e) {
		noProviderAbort = true;
	}
}

function App() {
	const [walAddress, setWalAddress] = useState('0x00');
	const [pctBal, setPctBal] = useState(0);
	const [ethBal, setEthBal] = useState(0);
	const [coinSymbol, setCoinSymbol] = useState("Nil");
	const [transAmount, setTransAmount] = useState('0');
	const [pendingFrom, setPendingFrom] = useState('0x00');
	const [pendingTo, setPendingTo] = useState('0x00');
	const [pendingAmount, setPendingAmount] = useState('0');
	const [isPending, setIsPending] = useState(false);
	const [errMsg, setErrMsg] = useState("Transaction failed!");
	const [isError, setIsError] = useState(false);

	// Aborts app if metamask etc not present
	if (noProviderAbort) {
		return (
			<div>
			<h1>Error</h1>
			<p><a href="https://metamask.io">Metamask</a> or equivalent required to access this page.</p>
			</div>
		);
	}

	// Notification to user that transaction sent to blockchain
	const PendingAlert = () => {
		if (!isPending) return null;
		return (
			<Alert key="pending" variant="info" 
			style={{position: 'absolute', top: 0}}>
			Blockchain event notification: transaction of {pendingAmount} 
			Eth from <br />
			{pendingFrom} <br /> to <br /> {pendingTo}.
			</Alert>
		);
	};

	// Notification to user of blockchain error
	const ErrorAlert = () => {
		if (!isError) return null;
		return (
			<Alert key="error" variant="danger" 
			style={{position: 'absolute', top: 0}}>
			{errMsg}
			</Alert>
		);
	};

	// Sets current balance of PCT for user
	signer.getAddress().then(response => {
		setWalAddress(response);
		return erc20.balanceOf(response);
	}).then(balance => {
		setPctBal(balance.toString())
	});

	// Sets current balance of Eth for user
	signer.getAddress().then(response => {
		return provider.getBalance(response);
	}).then(balance => {
		let formattedBalance = ethers.utils.formatUnits(balance, 18);
		setEthBal(formattedBalance.toString())
	});

	// Sets symbol of ERC20 token (i.e. PCT)
	async function getSymbol() {
		let symbol = await erc20.symbol();
		return symbol;
	}
	let symbol = getSymbol();
	symbol.then(x => setCoinSymbol(x.toString()));

	// Interacts with smart contract to buy PCT
	async function buyPCT() {
		// Converts integer as Eth to Wei,
		let amount = await ethers.utils.parseEther(transAmount.toString());
		try {
			await erc20.buyToken(transAmount, {value: amount});
			// Listens for event on blockchain
			await erc20.on("PCTBuyEvent", (from, to, amount) => {
				setPendingFrom(from.toString());
				setPendingTo(to.toString());
				setPendingAmount(amount.toString());
				setIsPending(true);
			})
		} catch(err) {
			if(typeof err.data !== 'undefined') {
				setErrMsg("Error: "+ err.data.message);
			} 
			setIsError(true);
		} 	
	}

	// Interacts with smart contract to sell PCT
	async function sellPCT() {
		try {
			await erc20.sellToken(transAmount);
			// Listens for event on blockchain
			await erc20.on("PCTSellEvent", (from, to, amount) => {
				setPendingFrom(from.toString());
				setPendingTo(to.toString());
				setPendingAmount(amount.toString());
				setIsPending(true);
			})
		} catch(err) {
			if(typeof err.data !== 'undefined') {
				setErrMsg("Error: "+ err.data.message);
			} 
			setIsError(true);
		} 
	}

	// Sets state for value to be transacted
	// Clears extant alerts
	function valueChange(value) {
		setTransAmount(value);
		setIsPending(false);
		setIsError(false);
	}

	// Handles user buy form submit
	const handleBuySubmit = (e: React.FormEvent) => {
		e.preventDefault();
		valueChange(e.target.buypct.value);
		buyPCT();
	};

	// Handles user sell form submit
	const handleSellSubmit = (e: React.FormEvent) => {
		e.preventDefault();
		valueChange(e.target.sellpct.value);
		sellPCT();
	};

	return (
		<div className="App">
		<header className="App-header">

		<ErrorAlert />
		<PendingAlert />

		<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/6f/Ethereum-icon-purple.svg/512px-Ethereum-icon-purple.svg.png" className="App-logo" alt="Ethereum logo" />

		<h2>{coinSymbol}</h2>

		<p>
		User Wallet address: {walAddress}<br/>
		Eth held: {ethBal}<br />
		PCT held: {pctBal}<br />
		</p>

		<form onSubmit={handleBuySubmit}>
		<p>
		<label htmlFor="buypct">PCT to buy:</label>
		<input type="number" step="1" min="0" id="buypct" 
		name="buypct" onChange={e => valueChange(e.target.value)} required 
		style={{margin:'12px'}}/>	
		<Button type="submit" >Buy PCT</Button>
		</p>
		</form>

		<form onSubmit={handleSellSubmit}>
		<p>
		<label htmlFor="sellpct">PCT to sell:</label>
		<input type="number" step="1" min="0" id="sellpct" 
		name="sellpct" onChange={e => valueChange(e.target.value)} required 
		style={{margin:'12px'}}/>	
		<Button type="submit" >Sell PCT</Button>
		</p>
		</form>

		<a  title="GitR0n1n / CC BY-SA (https://creativecommons.org/licenses/by-sa/4.0)" href="https://commons.wikimedia.org/wiki/File:Ethereum-icon-purple.svg">
		<span style={{fontSize:'12px',color:'grey'}}>
		Ethereum logo by GitRon1n
		</span></a>
		</header>
		</div>
	);
}

export default App;
```

## Deploy the contract

With everything built we are going to deploy the smart contract.  At the terminal we change directory back to the one containing our contract and deploy:

```bash
cd ..
truffle deploy
```

If everything works you should see output similar to this:

[![Truffle deploying PreciousChickenToken](https://www.preciouschicken.com/blog/images/truffle_deploy.png)](https://www.preciouschicken.com/blog/images/truffle_deploy.png)

I've highlighted the contract address in a yellow box on the above - this is the address on the blockchain that your contract has been deployed to (your address will be similar but different).

Now we know this address we have to change the client src code to reflect this.  Therefore edit *App.js*:

```bash
vim client/src/App.js 
```

find the relevant line of code, in my example it is: 

```javascript
const contractAddress ='0x3f45f12c12a6CCA5A0F0aA48Ec2214165FC6E7D0';
```

and replace the string within the single quotes with whatever the address you have noted down.

If you switch to Ganache you will see that the first account (Index 0) no longer has a balance of 100 Eth, this is because a small amount of Eth has been consumed in deploying the contract.  This account now also owns 1000 PreciousChickenTokens, although we don't know that looking at Ganache.

## Approve the transacting account

As the first account in Ganache (Index 0) is now set as the *owner* by the smart contract, e.g. it owns the 1000 ERC20 tokens; we'll be using the account at Index 1 to transact on.  The ERC20 specification says that when the [transferFrom](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#transferfrom) method is used authorisation has to be given specifically that Account A can pass Account B tokens.  We will do this using truffle console.  Therefore at the terminal:

```bash 
truffle console
```

Your prompt show now change to `truffle(ganache)>` or similar.  Enter:

```javascript
token = await PreciousChickenToken.deployed()
```

If successful this should return `undefined`.  We can now increase the allowance and then exit truffle console.

```javascript
token.increaseAllowance(accounts[1], 1000, {from: accounts[0]})
.exit
```

This code should output a transaction log to the console.  The command states that accounts[0] or the Ganache account at Index 0, the *owner*, authorises accounts[1] or Index 1, to hold up to 1000 tokens.

## Start React

Time to fire up React:

```bash
cd client
npm run start
```

Your browser should now point to `localhost:3000`.  If you do not have [Metamask](https://metamask.io) installed, or some equivalent software for accessing the Ethereum blockchain, then you will see the error message: *Metamask or equivalent required to access this page*.  The remainder of this guide will assume you have installed Metamask; I haven't tested other options (e.g. [Brave](https://brave.com/) browser), so they may not work.

If your browser is suitably enabled you should otherwise see the rotating Ethereum symbol, and a number of blank fields:

[![Pre-wallet import PreciousChickenToken splash screen](https://www.preciouschicken.com/blog/images/metamask_ff_blank.png)](https://www.preciouschicken.com/blog/images/metamask_ff_blank.png)

## Import wallet into Metamask

This process will assume your starting point is a fresh install of Metamask on Firefox (v77.0.1); other browsers are likely going to be different; and if you have already used Metamask previously you will need to logout etc.

Assuming you do have that fresh install selecting the Metamask extension icon will result in:

[![Pre-wallet import PreciousChickenToken splash screen](https://www.preciouschicken.com/blog/images/metamask_ff_welcome.png)](https://www.preciouschicken.com/blog/images/metamask_ff_welcome.png)

Selecting the *Get Started* option will take us to our set up options.  Here we want to select *No, I already have a seed phrase*:

[![Pre-wallet import PreciousChickenToken splash screen](https://www.preciouschicken.com/blog/images/metamask_ff_new.png)](https://www.preciouschicken.com/blog/images/metamask_ff_new.png)

We now need our seed phrase that Ganache has created for us; at the top of the screen we will find twelve words under the heading *Mnemonic*.  Copy and paste them into the seed phrase box below, and add a password of your choosing:

[![Pre-wallet import PreciousChickenToken splash screen](https://www.preciouschicken.com/blog/images/metamask_ff_import.png)](https://www.preciouschicken.com/blog/images/metamask_ff_import.png)

There will be a number of congratulatory / analytics sreens to click through after which you will see your account.  Currently blank as we haven't connected it to our local blockchain instance.  Therefore select *Networks* from the top right and then *Custom RPC*:

[![Pre-wallet import PreciousChickenToken splash screen](https://www.preciouschicken.com/blog/images/metamask_ff_network.png)](https://www.preciouschicken.com/blog/images/metamask_ff_network.png)


## Conclusions
