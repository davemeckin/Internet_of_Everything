# Putting it all Together

## Connecting your hardware to a realtime server and interacting with captured data using your React.js interface

This workshop is designed to make you think back to previous sessions so you can put all the stuff we've done together. If you are unsure, please check back through the workshops and the things we made to refresh your memory before asking for help.

Where code is supplied, please make every effort to type it out yourself, you will not learn by copying and pasting code.

### Task 1 - Pubnub History

- In the coursework brief, it states that you if you are using Pubnub, you must use the pubnub history API to store data captured. Find the coursework brief and read it thoroughly.
- Find the link to the pubnub tutorial in the coursework brief and follow the tutorial steps to enable the history API (you can only store history for 7 days with the free version).
- You might find [this link](https://support.pubnub.com/support/solutions/articles/14000043855-how-do-i-enable-add-on-features-for-my-keys-) useful


### Task 2 - Connect your Arduino to Pubnub

- Connect your Arduino to the ethernet shield.
- Connect a sensor to your Arduino. This could be any of the sensors we previously used: light detection/proximity detection/slide potentiometer).
- Write/load the necesary firmware onto your Arduino, you saved it from the 3rd Physical Computing workshop, right?
- Test your data flow works by checking your previous workshop page that was built in the third session of the Physical Computing workshops.


### Task 3 - Record Some Data

- Make sure you've been sending some data for some time, and make sure that data changes quite a lot over time.
- Disconnect your device from the pubnub connection.

### Task 4 - Ensure Your Data is Viewable 

- With the ```pubnub.history({})``` method that you experimented with in the tutorial, see if you can still view your data when you add this to the [page](ws3_MAKE_2020_Example.html) you built in the final session of the Physical Computing workshops.

- It might look something like this:

```javascript
let channel = 'YOUR CHANNEL NAME';

pubnub.history(
          {
        	channel : channel,
        	count : 36
        },
        function (status, response) {
            console.log(response.messages);
        }
    );
```
- And don't forget to set the History Flag to true in your eon chart:

```javascript


eon.chart({
        channels: [channel],
        history: true,
        flow: true,
        pubnub: pubnub,
        generate: {
          bindto: '#chart',
          data: {
            type: 'spline',
            labels: false
          }
        }
      });
```

### Task 5 - Working with React

- Make a new create-react-app project - remember how to do it? If not, go back to the first section of last week's tutorial.
- cd into your new project directory and type ```npm install pubnub```
- then we also want to install pubnub react: ```npm install pubnub-react@rc```
- In your app.js file, delete the contents as with the last tutorial, and create a blank app with a heading1 element showing a title such as "Realtime Sensor Data in ReactJS"

```javascript

import React from 'react';

class App extends React.Component {

  constructor(props) {
    super(props);
 

  }

  render() {
    return (
      <div className="App">
        <h1> Realtime Sensor Data in ReactJS </h1>
    
            </div>
    );
  }
}
export default App;
```


OK, now import pubnub using ```import PubNub from 'pubnub';``` at the top of your App.js file.
- Now make a new instance of pubnub in the ```constructor()``` of your app:

```javascript
this.pubnub = new PubNub({
        publishKey: 'YOUR PUBLISH KEY',
        subscribeKey: 'YOUR SUBSCRIBE KEY'
      });
```

- And finally in the constructor, we're going to set the state of our app using the state object:

```javascript
this.state = {
      chartType: 'spline',
      pubnub: this.pubnub,
      channel: 'ioechannel'

    };
```

### Task 6 - Using eon-react

- go to the terminal window.
- stop the development server by typing ctrl+c in the terminal window.
- ensure that you have cd'd into your root directory of your project (it should have defaulted to this location)
- we're going to install our Project Eon chart library to visualise our data, so type: ```npm install eon-react```

### Task 7 - Making an eon-chart Component

- Create a standard shell of a component with class structure - just as we did in the create-react-app tutorial last week. Call the class EonChart and save your file as EonChart.js in the src folder with your App.js file
- Now import react and eon so that we can use them in our component:
```javascript
import React from 'react';
import eon from 'eon-chart';
```

- OK we're going to use an in built React method called ```componentDidMount()``` to generate our eon chart:

```javascript
componentDidMount(){
    eon({
      pubnub: this.props.pubnub,
      channels: this.props.channels,
      history: true,
      generate: {
        bindto: '#chart',
        data: {
          labels: this.props.labels || true,
          type: this.props.type || 'line'
        }
      }
    });
    
  }
 ```

 - this is very similar to the one we made back in the 3rd Physical Computing session, however notice that we can now use props to set particular properties dynamically...

 - the ```||``` symbol in this case just helps us to define a default if one of the props is not set when the component mounted

 - And now let's just make sure our ```render()``` function returns a div with the id of "chart" to match what we have told eon to bind to:

 ```javascript
 render(){
    return(
      <div id="chart"> </div>
    );
  }
  ```

So, your final EonChart component class should look like this:

```javascript
import React from 'react';
import eon from 'eon-chart';

class EonChart extends React.Component{

  componentDidMount(){
    eon({
      pubnub: this.props.pubnub,
      channels: this.props.channels,
      history: true,
      generate: {
        bindto: '#chart',
        data: {
          labels: this.props.labels || true,
          type: this.props.type || 'line'
        }
      }
    });
    
  }

  render(){
    return(
      <div id="chart"> </div>
    );
  }
}

export default EonChart;
```

### Task 8 - Adding our EonChart component to our App

- OK let's go back to our App.js file.
- Now all we need to do is import our EonChart component like we would another library or class at the top of our App.js file:

```javascript
import EonChart from './EonChart.js';
```

- then, in the app div, beneath the heading, just add the EonChart component using JSX and set the props to our App state object settings:

```javascript
<EonChart
            pubnub={this.state.pubnub}
            channels={[this.state.channel]}
            type={this.state.chartType} />

```
So, your final App.js should look like this:

```javascript
import React from 'react';
import PubNub from 'pubnub';
import EonChart from './EonChart.js';
import './App.css';
 
class App extends React.Component {

  constructor(props) {
    super(props);
    
    this.pubnub  =  new  PubNub({
        publishKey: 'YOUR PUB KEY',
        subscribeKey:  'YOUR SUB KEY'
    });
  

    this.state = {
      chartType: 'spline',
      pubnub: this.pubnub,
      channel: 'ioechannel'

    };

  }

  

  render() {
    return (
      <div className="App">
        <h1> Realtime Sensor Data in ReactJS </h1>
             <EonChart
            pubnub={this.state.pubnub}
            channels={[this.state.channel]}
            type={this.state.chartType} />
      
   
            </div>
    );
  }
}
export default App;
```

***woohoo!*** you've managed to make your first realtime sensor data visualisation in react, pubnub and project Eon!

### Task 9 - Create some new components from material-ui examples

- Go to http://www.material-ui.com/ and check out some of their awesome ui components.
- Try copying their source code into new custom componenents of your own and add them to your project.
- I used the nav bar and the tool bar perhaps...




