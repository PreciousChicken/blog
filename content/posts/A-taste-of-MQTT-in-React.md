---
title: "A taste of MQTT in React"
date: 2020-03-03T21:43:50Z
tags: ["React", "MQTT"]
categories: ["Web development"]
description: "A very quick start to updating a page in React using MQTT"
draft: true
---

### Introduction

MQTT, is according to Wikipedia, "an open OASIS and ISO standard (ISO/IEC PRF 20922) lightweight, publish-subscribe network protocol that transports messages between devices."  It was first used to monitor an oil pipeline through the desert, and is now used in various Internet of Things scenarios.  This guide shows how you might update a [React](https://reactjs.org) page using a MQTT online broker - specifically [HiveMQ](https://www.hivemq.com).

### Prerequisites

-  [NodeJS](https://nodejs.org) - If you haven't installed it before, I found installing it using the Node Version Manager as suggested on this [Stack Overflow answer](https://stackoverflow.com/a/24404451/6333825) to cause less aggravation than downloading via the official website.
-  If you've previously installed `create-react-app` globally via `npm install -g create-react-app`, then uninstall it with the command `npm uninstall -g create-react-app` so you are using the latest version in the step below. 

### Use Create-React-App to initiate project

Use `create-react-app` to kick off our React project:

```bash
npx create-react-app mqtt_react
cd mqtt_react
```

### Install MQTT.js

[MQTT.js](https://github.com/mqttjs/MQTT.js) is a fully-featured Javascript library for the MQTT protocol.  Install as follows:

```bash
npm install mqtt
```

### Edit src/App.js

Open the file `src/App.js` using your favourite text editor, delete all the text and replace with:

```jsx
import React, { useState, Fragment } from 'react';
import './App.css';

var mqtt    = require('mqtt');
var client  = mqtt.connect('mqtt://broker.hivemq.com:8000/mqtt');
client.subscribe('preciouschicken.com');

function App() {
	var note;
	client.on('message', function (topic, message) {
		// message is Buffer
		note = message.toString();
		setMesg(note);
		console.log(note);
		client.end();
	});

	const [mesg, setMesg] = useState(<Fragment><em>nothing heard</em></Fragment>);

	return (
		<div className="App">
		<header className="App-header">
		<h1>A taste of MQTT in React</h1>
		<p>The message is: {mesg}</p>
		</header>
		</div>
	);
}

export default App;
```

### Start the app

Start the React application with:

```bash
npm start
```

and point your browser to [localhost:3000](http://localhost:3000) where you should see:

![Nothing heard](static/images/taste_of_react_nothing_heard.png)

### Publish a message via HiveMQ MQTT web broker

Head to the [HiveMQ MQTT web broker](http://www.hivemq.com/demos/websocket-client/) and in the _Connection_ section select the _Connect_ button (leaving the other defaults in place).  Now in the _Publish_ section change the _Topic_ field to `preciouschicken.com` and type any message you want in the _Message_ field.  Lastly select _Publish_; as illustrated:

![HiveMQ MQTT web broker](static/images/hivemq.png)

### See the results in the React app

Go back to [localhost:3000](localhost:3000) and you should see:

![Your message here](static/images/taste_of_react_your_message.png)

--- 

Congratulations.  You are a step closer to achieving your life's ambition of getting  your internet enabled fridge to post to the net...
