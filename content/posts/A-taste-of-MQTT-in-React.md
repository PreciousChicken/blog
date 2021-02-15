---
enableToc: true
title: "A taste of MQTT in React"
date: 2020-03-03T21:43:50Z
tags: ["React", "MQTT", "JavaScript", "Node"]
categories: ["Web development"]
description: "A very quick start to updating a page in React using MQTT"
draft: false
---

**Update Apr 20**: This tutorial previously used the MQTT online broker [HiveMQ](https://www.hivemq.com), which worked fine when you were using a local development server.  However once uploaded to a web site provider using HTTPS (pretty much everyone now), then it generated a mixed content error message.  This message was generated due to an insecure WebSocket (WS) protocol running underneath the secure HTTPS protocol, hence causing the browser to flag this.  The solution to this is to use the WebSocket Secure (WSS) protocol, which for some reason I could not get to work with HiveMQ.  This update therefore uses the [Eclipse Mosquitto MQTT broker](https://mosquitto.org), which allowed me to use WSS and fixed the issue.

### Introduction

MQTT, is according to Wikipedia, 
>"an open OASIS and ISO standard (ISO/IEC PRF 20922) lightweight, publish-subscribe network protocol that transports messages between devices."

It was first used to monitor an oil pipeline through the desert, and is now used in various Internet of Things scenarios.  This guide for the Linux command line shows how you might update a [React](https://reactjs.org) page using a MQTT online broker - specifically [Mosquitto](https://mosquitto.org).

For reference I'm using Ubuntu 18.04.4 LTS on the [Regolith](https://regolith-linux.org) desktop environment, mqtt.js 3.0.0, npm 6.14.4 and node 12.6.0.

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
var options = {
	protocol: 'mqtts',
	// clientId uniquely identifies client
	// choose any string you wish
	clientId: 'b0908853' 	
};
var client  = mqtt.connect('mqtt://test.mosquitto.org:8081', options);

// preciouschicken.com is the MQTT topic
client.subscribe('preciouschicken.com');

function App() {
  var note;
  client.on('message', function (topic, message) {
    note = message.toString();
    // Updates React state with message 
    setMesg(note);
    console.log(note);
    client.end();
    });

  // Sets default React state 
  const [mesg, setMesg] = useState(<Fragment><em>nothing heard</em></Fragment>);

  return (
    <div className="App">
    <header className="App-header">
    <h1>A taste of MQTT in React</h1>
    <p>The message is: {mesg}</p>
		<p>
		<a href="https://www.preciouschicken.com/blog/posts/a-taste-of-mqtt-in-react/"    
		style={{
			color: 'white'
		}}>preciouschicken.com/blog/posts/a-taste-of-mqtt-in-react/</a>
		</p>
		</header>
		</div>
  );
}

export default App;
```

(or alternatively download from [github](https://github.com/PreciousChicken/taste_of_mqtt_in_react)).


### Start the app

Start the React application with:

```bash
npm start
```

and point your browser to [localhost:3000](http://localhost:3000) where you should see:

[![Nothing heard](https://www.preciouschicken.com/blog/images/taste_of_react_nothing_heard-thumb.png)](https://www.preciouschicken.com/blog/images/taste_of_react_nothing_heard.png)

### Publish a message via the command line

Open the terminal and enter:

```bash
mqtt pub -t 'preciouschicken.com' -h 'test.mosquitto.org' -m 'Your message here!'
```

### See the results in the React app

Go back to [localhost:3000](http://localhost:3000) and you should see:

[![Your message here](https://www.preciouschicken.com/blog/images/taste_of_react_your_message-thumb.png)](https://www.preciouschicken.com/blog/images/taste_of_react_your_message.png)

The message should also appear in your browser's console.

In the event you see messages appearing that you haven't written, it means other people are using the same MQTT _Topic_ you are.  In that case remove the string `preciouschicken.com` from the file `src/App.js` and replace with something unique - for instance your birth year followed by your dog's name: e.g. `84rover`.  Use that same phrase in the _Topic_ flag (i.e. `-t`) of the MQTT command line and you will see your own messages only (assuming that other people born in 1984 with a dog named Rover aren't also broadcasting).

I've also uploaded the page to [taste-of-mqtt-in-react.preciouschicken.now.sh](https://taste-of-mqtt-in-react.preciouschicken.now.sh), where hopefully it can be demo'd live.

### Conclusion

Congratulations.  You are a step closer to achieving your life's ambition of getting  your internet enabled fridge to post to the net...

### Further reading

- [Beginner's Guide to the MQTT Protocol](http://www.steves-internet-guide.com/mqtt/)
- [MQTT.js documentation](https://www.hivemq.com/blog/mqtt-client-library-mqtt-js/) as part of HiveMQ's [MQTT Client Library Encyclopedia](https://www.hivemq.com/mqtt-client-library-encyclopedia/)
- [Getting Started with Node.js and MQTT](https://blog.risingstack.com/getting-started-with-nodejs-and-mqtt/)


