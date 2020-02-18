# Using React, p5 and OpenSoundControl (OSC)

## Connecting your hardware to communicate directly to another machine, interacting with captured data using react and p5...

This workshop is designed to make you think back to previous sessions so you can put all the stuff we've done together. If you are unsure, please check back through the workshops and the things we made to refresh your memory before asking for help.

Where code is supplied, please make every effort to type it out yourself, you will not learn by copying and pasting code.


### Task 1 - Connect your Arduino to Pubnub

- Connect your Arduino to the ethernet shield.
- Connect two sensors to your Arduino. This could be any of the sensors we previously used: light detection/proximity detection/slide potentiometer).
- Write/load the necesary firmware onto your Arduino, you saved it from the 5th Physical Computing workshop, right? (It's also on Blackboard)
- Test it is working by checking in the serial monitor window on the Arduino IDE


### Task 2 - Making a new app and installing the dependencies

- open a new terminal window on your mac. If you are using a UWE Bristol machine, you will probably have to run ```npm install -g create-react-app``` first.
- let's create a new basic react app shell using ```npx create-react-app reactoscp5```
- ```cd``` into your project folder, remember how to do this?
- OK, once in your project folder, we're going to install three different libraries to help us use react, p5 and OSC together...

- first of all, let's install the implementation of p5 in React with ```npm install react-p5```
- now, we need to install the OpenSoundControl implementation that p5 uses: ```npm install node-osc```
- finally, let's install the library that we use to connect our browser to the local server we need to have running on our machine for OSC to work: ```npm install socket.io```

- run ```npm start``` just to test that your basic app will compile. Ask for help if you have any issues. This will just run the default first create-react-app page at this point.

### Task 3 - Starting react-p5 and our local server

- So, the cool thing about react and npm is that basically someone has already made a library or component doing what we want to do before. We've already installed the react-p5 library but not only do they give us library, they give us some starter code too! Let's go grab it. Go to [this link](https://www.npmjs.com/package/react-p5) and copy the code from the "usage" section into the App.js file in your src folder. As with the previous tutorials, it's totally fine to delete the contents of App.js and replace them with this code. Save and see what happens...

- Do you see a white circle move off the screen? Take a look at the code and see if you can see what's going on. Hopefully some of it will seem familiar, and some of it may be a little different.

- Delete the line in the draw() function that moves the circle to the right.

- One final thing in this section, we want to steal the bridge.js file that acts as our realtime local server when run with node.js. I'm sure you remember this really well, but we needed this in Physical Computing session 5 when we were running OSC and p5 together to relay messages sent to the machine into the browser. 

- So, go to [this link](https://github.com/genekogan/p5js-osc), download the repo and copy the bridge.js file from that folder into your "reactoscp5" folder.

- Open a new terminal window, ```cd``` into the "reactoscp5" folder and run ```node bridge.js```

- Nothing will happen right now, but we're going to check back here later for messages about our connection.


### Task 4 - Updating our react-p5 sketch to prime it ready for some data

- at the top of App.js, we're going to import our socket.io library so that we can listen for data coming from our local server, on line 3 write: ```import socketIOClient from "socket.io-client";```

- Right, now you'll notice that the example code the good people who made react-p5 gave us doesn't contain a constructor. We need to change this, plus we're going create a state object to contain some initial settings and we're going to change x and y to this.x and this.y... this will allow us to access this.x and this.y from elsewhere in our code later. Remember where the constructor goes?

```javascript
constructor() {
    super();
    this.state = {
      endpoint: "http://127.0.0.1:8081", // this is telling our socket.io client to connect to our bridge.js node local server on port 8081
      oscPortIn: 7500, // this will configure our bridge.js node local server to receive OSC messages on port 7400
      oscPortOut: 3331 // this will configure our bridge.js node local server to send OSC messages on port 3331 (we're not actually sending anything in this sketch but it is required)
    };
    this.x = 50; // initial starting x point of our circle
    this.y = 50; // initial starting y point of our circle
  }
```

- now, in the draw function let's update our ellipse so that is uses this.x and this.y: ```p5.ellipse(this.x, this.y, 70, 70);```

### Task 5 - Connecting to our local server

- OK, not going to lie, the next bit is a bit fiddly. We're going to use the componentDidMount() function just like we did week. Then we're going to use pretty much the same code from examples provided with p5js-osc. 

- Beneath draw in your App.js file, add the following function (Please type this out so you can get a feel for what is going on in each line/block of code):

```javascript
componentDidMount() {
    const { endpoint } = this.state; // using our endpoint from state object
    const { oscPortIn } = this.state; // using our in port for OSC from state object
    const { oscPortOut } = this.state; // using our our port for OSC from state object
    const socket = socketIOClient(endpoint); // create an instance of our socket.io client
    socket.on('connect', function() { // connect and configure local server with settings from the state object
        socket.emit('config', { 
          server: { port: oscPortIn,  host: '127.0.0.1'},
          client: { port: oscPortOut, host: '127.0.0.1'}
        });
      });
      socket.on('message', (function(msg) { // once we receive a message, process it a bit then call this.receiveOSC()
        if (msg[0] === '#bundle') { // treat it slightly differently if it's a bundle or not
          for (var i=2; i<msg.length; i++) {
            this.receiveOsc(msg[i][0], msg[i].splice(1));
          }
        } else {
          this.receiveOsc(msg[0], msg.splice(1));
        }
      }).bind(this)); // we have to explicitly bind this to the upper execution context so that we can call this.receiveOSC()
  }
```
- You may get an error at this stage about this.receiveOSC() not being a function, that's going to be our next task!

### Task 6 - Getting the OSC in and controlling our circle

The final thing in our app code that we need to do is add the function that actually assigns this.x and this.y to our incoming data:


```javascript
 receiveOsc(address, value) {
      console.log("received OSC: " + address + ", " + value);

      if (address == '/analogue') {
        this.x = value[0];
        this.y = value[1];    
      }
      
  }
```

Your final App.js file should look like this"

```javascript
import React, { Component } from "react";
import Sketch from "react-p5";
import socketIOClient from "socket.io-client";
 
export default class App extends Component {
  

  constructor() {
    super();
    this.state = {
      endpoint: "http://127.0.0.1:8081",
      oscPortIn: 7500,
      oscPortOut: 3331
    };
    this.x = 50;
    this.y = 50;
  }
 
  setup = (p5, canvasParentRef) => {
    p5.createCanvas(p5.windowWidth, p5.windowHeight).parent(canvasParentRef); // use parent to render canvas in this ref (without that p5 render this canvas outside your component)
   
  };
  draw = p5 => {
    p5.background(0);
    p5.ellipse(this.x, this.y, 70, 70);
    // NOTE: Do not use setState in draw function or in functions that is executed in draw function... pls use normal variables or class properties for this purposes
    //this.x++;
  };

  componentDidMount() {
    const { endpoint } = this.state;
    const { oscPortIn } = this.state;
    const { oscPortOut } = this.state;
    const socket = socketIOClient(endpoint);
    socket.on('connect', function() {
        socket.emit('config', { 
          server: { port: oscPortIn,  host: '127.0.0.1'},
          client: { port: oscPortOut, host: '127.0.0.1'}
        });
      });
      socket.on('message', (function(msg) {
        if (msg[0] === '#bundle') {
          for (var i=2; i<msg.length; i++) {
            this.receiveOsc(msg[i][0], msg[i].splice(1));
          }
        } else {
          this.receiveOsc(msg[0], msg.splice(1));
        }
      }).bind(this));
  }

  receiveOsc(address, value) {
      console.log("received OSC: " + address + ", " + value);

      if (address == '/analogue') {
        this.x = value[0];
        this.y = value[1];
      }
      
  }

  
 
  render() {
 
    return (
      <div>
        <Sketch setup={this.setup} draw={this.draw} />
      </div>
      );
  }
}
```

***woohoo!*** you've managed to make your first realtime sensor data visualisation in react, p5 and OSC!

### Task 7 - Make your sketch look çøø¬

- Your p5 skills are now tightly honed so you are in a position to make a cool sketch. Can you port the code from the 5th Physical Computing session for instance?

