# Putting it all Together

## Connecting your hardware to a realtime server and interacting with captured data using your React.js interface


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
- In your app.js file, import

### Task 6 - Installing and Importing c3.js

- cd into your new project directory and type ```npm install c3```
- go into your node_modules folder and check for the folder.

### Task 7 - Making a c3 Chart Component

- Create a standard shell of a component with class structure - just as we did in the create-react-app tutorial last week.
- Now import c3 using ```import c3 from 'c3';```
- Make sure you also import the css using ``` 

### Task 8 - Create some new components from material-ui examples

- Go to http://www.material-ui.com/ and check out some of their awesome ui components.
- Try copying their source code into new custom componenents of your own and add them to your project.
- NOTE: all material ui components need to be enclosed in <MuiThemeProvider></MuiThemeProvider> tags.

