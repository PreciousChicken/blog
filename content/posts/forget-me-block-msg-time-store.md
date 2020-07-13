---
title: "Forget-me-Block: Message Time Store"
date: 2020-07-13T15:21:35+01:00
tags: ["ethereum", "blockchain", "solidity" ]
categories: ["Research"]
description: "Research into using the Ethereum blockchain for data preservation."
draft: false
---

## Research Aim

As part of ongoing research into using the Ethereum blockchain for the purpose of data preservation strategies, the aim is to create the following deliverables:

- A smart contract that provides notification of previously recorded information, at a predetermined future date, or prompted by a retrieval request.
- A DApp (Decentralised Application) based around the smart contract above where the user is notified via an interface such as a web browser.

This post is intended as a summary of this research deliverable.

## Challenges

The main obstacle in alerting the user at a predetermined future data, is that there is no notification function native to Ethereum, to quote the [documentation](https://solidity.readthedocs.io/en/v0.6.11/contracts.html#contracts):

> A contract and its functions need to be called for anything to happen. There is no “cron” concept in Ethereum to call a function at a particular event automatically.

That leaves the developer with a number of choices:

- [Ethereum Alarm Clock](https://www.ethereum-alarm-clock.com).  A decentralised network which financially motivates participants to run "TimeNodes." These nodes compete to monitor the Ethereum blockchain for time instances where smart contacts should be executed.  Should the node win the right to execute the smart contract, they are rewarded in Ether.  Unfortunately at time of writing however, although this system could be used for scheduling Ether transactions, there appeared little documentation on how this could be integrated within third party smart contracts.
- [Aion](https://www.aion.ethpantheon.com/).  A third party offering a service to execute smart contracts at a predetermined time in the future, in return for payment.  Advertises it can be "easily integrated into other smart contracts using a single scheduling function."
- Client side.  The developer takes responsibility for implementing a mechanism to check the smart contract at a suitable date-time within the client (e.g. web page, smartphone application) that the user interfaces the smart contract.

For the purposes of this deliverable the client side method has been chosen.

## Solution

### Smart contract

A smart contract satisfying the aim has been deployed to the Ropsten test network at address [0x2c93564045502dBC5e90009e8408C0B7b888523e](https://ropsten.etherscan.io/address/0x2c93564045502dBC5e90009e8408C0B7b888523e).  It has two externally accessible functions:

- *storeMsg(string memory _storedMsg, uint _unlockTime)*.  The two arguments are the message to be stored, and the time (using unix time) it needs to be stored until.  Messages are mapped against *msg.sender* - i.e. the users address.  This means that Alice can only retrieve messages that Alice has submitted, while Bob will only see messages he has submitted.  Eve will see neither. 
- *getMsgTimed() public view returns (StoredData[] memory)*.  This returns an array of objects containing all messages which have passed the *unlockTime* defined by the user.  The *pragma experimental ABIEncoderV2* option has been used within the smart contract to allow objects to be returned.  Solidity is notable in that although *state* arrays can be dynamic, *memory* arrays cannot be and have to be a fixed length.  Resultantly when the function is called and messages exist that have not yet reached unlock time, the returned array will contain empty values for the yet unlocked messages.  The single-page-application checks for empty values (specifically *id*) prior to presenting to the user, to ameliorate this.

### DApp

A web application, written in React, has been published at [forget-me-block-msg-time-store.preciouschicken.com](https://forget-me-block-msg-time-store.preciouschicken.com).  This single-page-application allows users to submit messages whilst deciding how long they wish to keep them 'locked' for.  It will then display unlocked messages (i.e. those which have been stored for the selected duration) on screen either a) automatically checking at 5 minute intervals or b) at user request.

Users will require a suitable in browser wallet (e.g. [Metamask](https://metamask.io)) and Ropsten test network Ether, available from the [Ropsten Ethereum Faucet](https://faucet.ropsten.be).

## Codebase

All code used is available at [PreciousChicken/forget-me-block-msg-time-store](https://github.com/PreciousChicken/forget-me-block-msg-time-store).

## Backlog

There is additional functionality that would improve this DApp:

- Ability for user to delete messages they have viewed.  Currently users can append messages to their store, but cannot remove them.
- Use an *event* within smart contract to confirm to user (ideally via a toast) that their message is being stored on the blockchain

For reasons of time compression the following has been minimised:

- Test coverage
- Smart contract optimisation (specifically ensuring the least Ether possible is used)
