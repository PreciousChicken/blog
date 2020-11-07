---
title: "Forget-me-block: Ethereum Calendar"
date: 2020-09-01T15:06:49+01:00
tags: ["Ethereum", "blockchain", "solidity", "research", "calendar" ]
categories: ["Research"]
description: "Research into using the Ethereum blockchain for data preservation."
draft: false
---

## Introduction

As we progress through life's journey we lose digital information.  Who we met, what we talked about, contact details, etc tend to disappear in the digital flux like tears in the rain.  This loss is often due to reasons of software (e.g. file formats changes) and hardware (e.g. corrupt hard drives).  But more prosaically organisational change happens: as we switch providers from Big Tech Corp A to Big Tech Corp B and back again, we often lose access to data we have stored in someone else's stovepipe.

This post details a proof of concept to combat this by hosting a calendar as an Ethereum smart contract.  The [eth-cal-open](#eth-cal-open) calendar implementation allows read-write access via an Ethereum enabled (e.g. metamask) web browser; it also allows a user to access a read-only version of the calendar in a standard calendar client (e.g. MS Outlook).  The theoretical advantage here is that as long as the Ethereum network exists you will be able to access your data.

This proof of concept has also been expanded into a role based access version, [eth-cal-auth](#eth-cal-auth), where an administrator (which defaults to the smart contract owner) allocates other users (as represented by Ethereum accounts) with date-ranged read or write access.  Typical use cases include an organisational group calendar.  The primary difference between this and today's solutions is that once access has been granted (and provided it is not rescinded) a user's access will continue regardless of their relationship to the organisation - or indeed whether the organisation continues to exist.

This post is intended to present a synopsis, links to repositories and running instances of deliverables from my recent MSc Thesis which explore this concept: *Forget-me-block - Exploring digital preservation strategies using Distributed Ledger Technology in the context of personal information management*.

## Thesis executive statement

Received wisdom portrays digital records as guaranteeing perpetuity; as the [New York Times wrote](https://www.nytimes.com/2010/07/25/magazine/25privacy-t2.html) a decade ago: "the web means the end of forgetting".  The reality however is that digital records suffer similar risks of access loss as the analogue versions they replaced - but through the mechanisms of software, hardware and organisational change.

The first two of these mechanisms are straightforward.  Software change relates to how data is encoded - for instance later versions of Microsoft Word often cannot access documents written with earlier versions.  Likewise hardware formats obsolesce; even popular technologies such as the floppy disk reach a point where accessing data on these formats becomes increasingly difficult.

The third mechanism is however more abstract as it relates to societal structures, and ironically is often generated as a by-product of attempts to escape the first two risks.  In our efforts to rid ourselves of hardware and software change these risks are often delegated to specialised external parties.  Common use cases are those of conveying information to a future self, e.g. calendars, diaries, tasks, etc.  These applications, categorised as Personal Information Management (PIM), negate the frailty of human memory.  Frequently these are outsourced at two removes - firstly by the individual to their employer (e.g. using a company system) and then by their employer to an external provider.  So enters organisational change risk; by the time the information is required the organisational chain that links user to data may be broken: the employer will have moved to a different provider, the employee will have left the company, the IS provider will be out of business or pivoted to new offerings.  

The wholesale outsourcing of these responsibilities, also means we do not own our digital identities. The current technological world sees organisations create our identity for us - whether that is our employer, tech giant or social media empire.  As we move between organisations, we also have to relinquish the identities that those organisations created for us, and with it the data that belongs to those identities. 

This research examines this idea - and asks whether blockchain might be able to challenge these norms.  The form of PIM chosen to implement was a calendar - everyone uses them and they are nice to demonstrate with as you can use them in a web browser or email client (e.g. MS Outlook).  In the course of this research a number of software deliverables were created:

- **CalStore**.  A smart contract which stores calendar events, with each address that calling the contract being allocated their own calendar.  On request it returns a calendar of events for the calling address using either the *iCal* format (see [RFC 5545](https://tools.ietf.org/html/rfc5545)) or JSON.
- **CalAuth**.  This smart contract is meant to represent an organisation (e.g. Blankshire General Hospital) who maintains a group calendar on CalStore.  CalAuth lets the contract owner authenticate which accounts are allowed to read and write to the group's calendar on CalStore.  All the functions which relate to creating and reading events have the same arguments and returns as CalStore: when an Externally Owned Account calls CalAuth - assuming they are not the admin and have the required access) then CalAuth essentially acts as an interface or relay to CalStore.  This is illustrated at the Architecture section below.
- **Eth-cal-open**.  A web calendar and API (generating an *iCal* feed to be used within a calendar client such as MS Outlook), this is a front end to *CalStore*.
- **Eth-cal-auth**.  This view depends on your account's level of access.  The contract owner will see an administrator's dashboard (where they can authenticate others), an authenticated user will see a web calendar featuring the group calendar (and API link) and an unauthenticated user will be denied access.

The artefacts above seek to demonstrate that there may be alternatives to the current norm - alternatives that give ownership of digital identities back to the individual. Within the artefacts once you have been granted access to an organisation’s data within a certain date range – unless the organisation specifically forbids you – then you will continue to have access to that data regardless of your association with that organisation. Indeed the organisation could cease to exist, yet your access to that data, would live on unaffected. This is a starkly different model to the current norm.

## Architecture

The UML use case diagram below illustrates that both Alice and Bob maintain accounts directly on *CalStore*, they also both have access (though at different levels) to a group calendar maintained by *shef.ac.uk*.  Bob also has access to a group calendar maintained by *bgh.nhs.uk*.

[![Eth-cal: UML Use Case diagram](https://www.preciouschicken.com/blog/images/ethcal-Use_Case_Architecture.png)](https://www.preciouschicken.com/blog/images/ethcal-Use_Case_Architecture.png)

The following UML sequence diagram shows the processes involved with Bob reading and writing to a *CalAuth* and *CalStore* contract.

[![Eth-cal: UML Sequence diagram](https://www.preciouschicken.com/blog/images/ethcal-Sequence.png)](https://www.preciouschicken.com/blog/images/ethcal-Sequence.png)

## Eth-cal-open

This instance is live on the Ropsten test net and will accept a connection from any account.

### Live instance
[forget-me-block-eth-cal-open.preciouschicken.com](https://forget-me-block-eth-cal-open.preciouschicken.com/)

### GitHub repo
[forget-me-block-eth-cal-open](https://github.com/PreciousChicken/forget-me-block-eth-cal-open)

### Screenshot - Calendar
[![Eth-cal-open](https://www.preciouschicken.com/blog/images/ethcalopen-view.png)](https://www.preciouschicken.com/blog/images/ethcalopen-view.png)

## Eth-cal-auth

This instance is live on the Ropsten test net and the view, as shown in screenshots below, will differ depending on whether you have administrator, read-write, read-only or nil access.

### Live instance
[forget-me-block-eth-cal-auth.preciouschicken.com](https://forget-me-block-eth-cal-auth.preciouschicken.com/)


### GitHub repo
[forget-me-block-eth-cal-auth](https://github.com/PreciousChicken/forget-me-block-eth-cal-auth)

### Screenshot - Admin dashboard

The view presented to those with administrator access (which defaults to smart contract owner):

[![Eth-cal-auth: Dashboard](https://www.preciouschicken.com/blog/images/ethcalauth-dashboard.png)](https://www.preciouschicken.com/blog/images/ethcalauth-dashboard.png)

### Screenshot - Calendar

The view presented to those with read-write or read-only access (the access level being displayed to the user next to their Ethereum account address):

[![Eth-cal-auth: Calendar](https://www.preciouschicken.com/blog/images/ethcalauth-view.png)](https://www.preciouschicken.com/blog/images/ethcalauth-view.png)

