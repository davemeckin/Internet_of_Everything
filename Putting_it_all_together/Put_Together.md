# Putting it all Together

## Connecting your hardware to a realtime server and interacting with captured data using your React.js interface

This workshop is designed to make you think back to previous sessions so you can put all the stuff we've done together. If you are unsure, please check back through the workshops and the things we made to refresh your memory before asking for help.

### Task 1 - Pubnub History

- In the coursework brief, it states that you must use the pubnub history API to store data captured. Find the coursework brief and read it thoroughly.
- Find the link to the pubnub tutorial in the coursework brief and follow the tutorial steps to enable the history API.


### Task 2 - Connect your Arduino to Pubnub

- Connect your Arduino to the ethernet shield.
- Connect a sensor to your Arduino. This could be any of the sensors we previously used: light detection/proximity detection/slide potentiometer).
- Write/load the necesary firmware onto your Arduino.
- Test your data flow works by checking your previous workshop page that was built in the final session of the Physical Computing workshops.


### Task 3 - Record Some Data

- Make sure you've been sending some data for some time, and make sure that data changes quite a lot over time.
- Disconnect your device from the pubnub connection.

### Task 4 - Ensure Your Data is Viewable 

- With the ```pubnub.history({})``` method that you experimented with in the tutorial, see if you can still view your data when you add this to the page you built in the final session of the Physical Computing workshops.

- It might look something like this:

```javascript
pubnub.history(
          {
        	channel : 'YOUR CHANNEL NAME',
        	count : 36
        },
        function (status, response) {
            console.log(response.messages);
        }
    );
```
### Task 5 - Working with React

- Make a new create-react-app project - remember how to do it? If not, go back to the first section of last week's tutorial.
- cd into your new project directory and type ```npm install pubnub```
- In your app.js file, import pubnub using ```import PubNub from 'pubnub';``` at the top of your App.js file.
- Now make a new instance of pubnub in the constructor of your app:

```javascript
this.pubnub = new PubNub({
        publishKey: 'YOUR PUBLISH KEY',
        subscribeKey: 'YOUR SUBSCRIBE KEY'
      });
```

- still in the constructor, make a new 2D array, something like this:

```javascript
this.datepub = [[],[]];
```

- now, we're going to read our pubnub history into our array. I'm using 1 dimensional data so for practice purposes, I'm just going to reverse the same data and stick it in the second slot. We also need to bind the our response function to the scope of the App class:

```javascript
this.pubnub.history(
          {
        channel : 'iotchannel',
        count : 360
        },
        (function (status, response) {

          for(let i = 0; i < response.messages.length; i++) {
            this.datepub[0][i] = response.messages[i]['entry']['eon']['sensor']; //reading response messages into array
          }

          this.datepub[1] = this.datepub[0].slice().reverse();	// reversing second array
          this.datepub[0].unshift('data1'); // adding label to start of array 0
          this.datepub[1].unshift('data2'); // adding label to start of array 1
        }).bind(this)						//binding to present execution context
      );
```

- And finally in the constructor, we're going to set the state of our app using the state object:

```javascript
this.state = {
      chartType: 'spline',
      datepub: this.datepub,
      pubnub: this.pubnub,
      channel: 'iotchannel'

    };
```

- Within our App class, we also need to make two functions to change the state of our App and pass those as props down to the Chart component that we're going to make in the next two stages:

```javascript
  _setBarChart = () => {
        //console.log("yoBar");
        this.setState({ chartType: 'bar' });
    }

    _setLineChart = () => {
        //console.log("yoLine", this.setState);
        this.setState({ chartType: 'spline' })
    }
```

 Try firing up your updated component and printing ```this.datepub``` to the console. Can you see some data in there? If not, go back through and try and debug to see where things went awry.

### Task 6 - Installing and Importing c3.js

- cd into your new project directory and type ```npm install c3```
- go into your node_modules folder and check for the folder.

### Task 7 - Making a c3 Chart Component

- Create a standard shell of a component with class structure - just as we did in the create-react-app tutorial last week. Call the class Chart and save your file as Chart.js
- Now import c3 using ```import c3 from 'c3';```
- Make sure you also import the css using ```import '../node_modules/c3/c3.css';``` 
- OK we're going to use an in built React method called ```componentDidMount()``` to generate our c3 chart:

```javascript
componentDidMount() {
    //console.log(this.props.datepub);
     this.chart = c3.generate({
      bindto: '#'+this.props.ident,
      data: { 
        columns: this.props.datepub,
        type: this.props.chartType,
        colors: {
            data1: '#CFD8DC',
            data2: '#673AB7'
        }
      }
    });
  }
 ```
 - And now let's add a method to change the chart type and update the data dynamically. Your final class definition should look something like this:

 ```javascript
 class Chart extends Component {

 
  componentDidMount() {
    
     this.chart = c3.generate({
      bindto: '#'+this.props.ident,
      data: { 
        columns: this.props.datepub,
        type: this.props.chartType,
        colors: {
            data1: '#CFD8DC',
            data2: '#673AB7',
            data3: '#0000ff'
        }
      }
    });
  }

  componentDidUpdate() {
    this._updateChart();
  }
  _updateChart() {

    this.chart.load({columns: this.props.datepub, type: this.props.chartType});
    
  }
  render() {
    return <div id={this.props.ident}></div>;    
  }
}

Chart.defaultProps = {
  chartType: 'spline'
}

export default Chart;
```
- Now, back in your App class, import your Chart component somewhere at the top.
- Then in the main app wrapper div, add your chart, setting the relevent props from the App state:

```javascript
<Chart ident="chart1" datepub = {this.state.datepub} pubnub={this.state.pubnub} channel={this.state.channel} chartType={this.state.chartType}/>
```

- OK now add two buttons which call the ```_setLineChart()``` and ```_setBarChart()``` methods. HINT: This works exactly the same as HTML using the onClick method attribute!

### Task 8 - Create some new components from material-ui examples

- Go to http://www.material-ui.com/ and check out some of their awesome ui components.
- Try copying their source code into new custom componenents of your own and add them to your project.
- NOTE: all material ui components need to be enclosed in <MuiThemeProvider></MuiThemeProvider> tags.
