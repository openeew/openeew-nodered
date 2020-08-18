# Node-RED Open Earthquake Early Warning Alert Map
Build a Earthquake Early Warning Alert Map Node-RED Dashboard using OpenEEW data

Learn how to build an Earthquake Early Warning system using live [OpenEEW](http://openeew.com) sensor data and visualize
historical seismic datasets in Node-RED Dashboards.

These example flows and Node-RED Dashboards might be useful as part of an Earthquake Early Warning system.
Join the Call for Code challenge and contribute to open source projects.

![OpenEEW Earthquake animation](images/mx72mag.gif?raw=true "OpenEEW graph animation")

### Examples Index

- [Install Node-RED and the prerequistes required to build the dashboards](#prerequistes)
- [Learn how to program an algorithm to detect seismic shaking](#seismic-activity-algorithm)
- [Construct a dashboard that displays live Earthquake Early Warning Sensor Alerts on a map and sends a SMS warning if a possible earthquake is detected](#example1)
- [Construct a dashboard that plots real time seismic activity sensor graphs in a chart by subscribing to a MQTT broker](#example2)
- [Construct a dashboard that plots the historical seismic activity from an OpenEEW dataset](#example3)

### Prerequistes

- [Install Node-RED](https://nodered.org/docs/getting-started/) on your system or in the cloud
  - This flow can be deployed to [IBM Cloud](https://developer.ibm.com/dwwi/jsp/register.jsp?eventid=cfc-2020-projects) by creating a [Node-RED Starter Application](https://developer.ibm.com/components/node-red/tutorials/how-to-create-a-node-red-starter-application/)
- [Add the following nodes](https://nodered.org/docs/user-guide/runtime/adding-nodes) to your Node-RED palette
  - [node-red-dashboard](https://flows.nodered.org/node/node-red-dashboard)
  - [node-red-contrib-web-worldmap](https://flows.nodered.org/node/node-red-contrib-web-worldmap)
  - [node-red-node-twilio](https://flows.nodered.org/node/node-red-node-twilio)
  - [node-red-node-ui-table](https://flows.nodered.org/node/node-red-node-ui-table)

  #### Local installation instructions

  ```
  npm install node-red-dashboard node-red-contrib-web-worldmap node-red-node-twilio node-red-node-ui-table
  ```

  #### IBM Cloud installation instructions

  - Instead of adding the additional packages via **Manage Palette**, use the IBM Cloud Toolchain and the git repository in IBM Cloud to add the following packages to the package.json. Commit the change and the CI/CD toolchain will restage the CF application.

  ```
  "node-red-dashboard":"2.x",
  "node-red-contrib-web-worldmap":"2.x",
  "node-red-node-twilio":"0.x",
  "node-red-node-ui-table":"0.x",
  ```    

### Seismic Activity algorithm

One simplistic approach to detecting seismic activity is to measure large accelerations from the openeew accelerometer sensors, based on a set threshold that is above the typical noise floor of the accelerometer. Wikipedia defines a [gal (unit)](https://en.wikipedia.org/wiki/Gal_(unit)) as a unit of acceleration used extensively in the science of gravimetry. The gal is defined as 1 centimeter per second squared (1 cm/s2). Seismic activity of interest might exceed 3 gals (3 cm/sec2).

These Node-RED flows observe the real-time openeew accelerometer data and calculate if the sensor might be experiencing seismic activity using the following algorithm. Each second this function receives x/y/z arrays of subsecond vibration data.  The data arrays are passed into the function inside ```msg.payload.traces[0]```  The javascript function loops through the vibration data looking for acceleration exceeding 3 cm/sec2

```javascript
// The simplest way to detect changes :
// "not earthquake" vs "possible earthquake"
// For each tuple x,y,z take the
// square root of sum of the squared of each components:
// (x**2 + y**2 + z**2)**(1/2)
// square rooting gives back positive values

// Loop through all of the subsecond arrays of data
// Most of this is noise
// 1 cm/sec2 == 1 gal
// +/- 0.3 gals is jitter

var alert = false;
var gals = 0.0;
var x,y,z;
for( i=0; i<msg.payload.traces[0].x.length;i++){
    x = msg.payload.traces[0].x[i];
    y = msg.payload.traces[0].y[i];
    z = msg.payload.traces[0].z[i];

    gals = Math.sqrt( Math.pow(x,2) + Math.pow(y,2) + Math.pow(z,2) );
    if( gals > 3 ) {
        // whoa - maybe an earthquake
        alert = true
    }
}

if( alert ) {
    return msg;
} else {
    return null ;
}
```

## Node-RED flow examples in this repository:

Learn how to implement OpenEEW Node-RED dashboards using these example flows.

Four examples are provided in the [flows](https://github.com/openeew/openeew-nodered/tree/master/flows) folder.

---
<a name="example1"></a>
### Example 1 : A flow that displays Earthquake Early Warning Sensor Alerts on a map

This flow plots the OpenEEW sensors on a map of Mexico and displays their seismic activity status.
The flow subscribes to the live data feed of the sensors using MQTT.
If any of the sensors have not checked in for 30 seconds mark the sensor offline.
If the seismic algorithm detects shaking, mark the sensor in red and send a Twilio SMS alert to warn residents to seek safety.

![OpenEEW Alert Dashboard](images/openeew-quakemap-v1-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-sensorstatus.json">Node-RED flow for OpenEEW Alerts</strong></a>
</p>

![OpenEEW Sensor Status flow](images/openeew-sensorstatus-flow.png?raw=true "OpenEEW flow")

This flow has four sections:
1. The **One time - Add Sensor pins to Map** section drops pins on the terrain map at the latitude / longitude locations of the Grillo OpenEEW Earthquake Early Warning System sensor network.
1. The **Report Sensor Status** section periodically checks if the sensors have recently reported, via MQTT, a seismic reading. If a sensor has not been seen during the last cycle, mark the device offline by changing the dropped pin to black with a warning symbol. When the sensor reconnects and comes back online, mark the device green on the map.
1. The **Subscribe to OpenEEW Sensor Network / Monitor Sensor Status** section subscribes to the MQTT broker and records the timestamps of the devices as they report seismic readings.  Details about the MQTT broker credentials need to be acquired in the [Slack workspace](https://join.slack.com/t/openeew/shared_invite/zt-cibhc0za-XKReMPobi2DsrPusORJZVQ).
1. The **Listen for possible Earthquakes** section runs the seismic activity shake algorithm described above.  If a possible earthquake is detected, send a Twilio alert and mark the dropped pin red on the map.  You will need to configure the Twilio SMS node with your [Twilio account](https://www.twilio.com/) details.

---
<a name="example2"></a>
### Example 2 : A flow that plots near real time Seismic activity sensor graphs in a chart

This flow subscribes to the live data feed (available via MQTT) of a selected sensor and plots the seismic activity in a set of
X / Y / Z graphs.

![OpenEEW Sensor Dashboard](images/openeew-sensorplot-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-sensorplot.json">Node-RED flow for OpenEEW Sensor graphs</strong></a>
</p>

![OpenEEW Sensor flow](images/openeew-sensorplot-flow.png?raw=true "OpenEEW flow")

This flow has three sections:
1. The **Enable / Disable Seismic Sensor Charts** section saves the state of the Switch node that determines if the sensor data should be plotted.
1. The **Select an OpenEEW Sensor Network sensor to plot** section loads a table of OpenEEW sensors and allows the investigator to choose which sensor to plot.
1. The **Plot selected real-time sensor data** section subscribes to a MQTT broker and filters data from the selected sensor. It then splits the live sensor data into X / Y / Z coordinates and plots 15 accelerometer readings per second to the Node-RED Dashboard chart nodes.  For performance reasons, depending on the sensitivity of the sensor, it discards the remaining accelerometer data.    

---
<a name="example3"></a>
### Example 3 : Select a Sensor and time range, plot the historical seismic activity from a OpenEEW dataset

This flow displays a Node-RED dashboard which presents the investigator / seismologist with a calendar,
time interval options and a list of OpenEEW sensors. The investigator selects an interesting sensor
and time period to study and queries the OpenEEW dataset. The flow then plots the historical sensor
data in a set of graphs.

The flow constructs a URL for the selected sensor and time interval and retrieves the historical sensor data from the
[OpenEEW dataset](https://s3.amazonaws.com/grillo-openeew/index.html#records/)
and plays it back into a graph in a Node-RED Dashboard.  

The "Next" button in the Historical Quake Playback Node-RED dashboard is incredibly insightful. The focus is often on the big earthquakes, and their specific timestamp / dataset.  The "Next" button lets the investigator explore the numerous aftershocks.  Start with the Mexico 7.2 magnitude earthquake on Feb 16 2018 23:35 and its data set and then click the Next button to watch (in 5 minute increments) the aftershocks that occurred over the next several hours.  Watch the animated gif of the earthquake and aftershock activity.

You can also observe the seismic activity by using the AutoPlay feature. Turn on the AutoPlay switch and select the number of minutes of playback.

The screenshot is of a
[magnitude 7.2 earthquake in Mexico](https://blog.grillo.io/analyzing-a-magnitude-7-2-earthquake-in-mexico-using-python-6272a4ff63e3) on
[2018-02-16 23:43:00](https://s3.amazonaws.com/grillo-openeew/index.html#records/country_code=mx/device_id=006/year=2018/month=02/day=16/hour=23/40.jsonl).

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplaybackv5-dashboard.png?raw=true "OpenEEW Dashboard")

Here is another screenshot of a
magnitude 4.1 earthquake in Puerto Rico on
[2020-06-01 12:05:51](https://s3.amazonaws.com/grillo-openeew/index.html#records/country_code=pr/device_id=3ef3d787af85/year=2020/month=06/day=01/hour=12/05.jsonl)

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplaybackv3-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-quakemap-v5.json">Node-RED flow for OpenEEW Historical Playback</strong></a>
</p>

![OpenEEW Historical Playback Flow](images/openeew-quakeplaybackv5-flow.png?raw=true "OpenEEW flow")

This flow has six sections:
1. The **Select a Historical Date** section displays a Node-RED Dashboard which allows the earthquake investigator to select a date from a Calendar widget, drop down widgets to select the hour and 5 minute segment, an option to AutoPlay the data for several minutes. These selections are stored in flow variables.
1. The **Build / Display OpenEEW dataset to retrieve** section takes the selected dates and constructs the dataset file to be downloaded / plotted.
1. The **Select a Sensor to plot** section lists a table of sensors and their location coordinates.
1. The **Next dataset** section provides a Next button and AutoPlay option to cycle through a timeseries of adjacent datasets. It calculates the name of the next dataset to be retrieved (handling all edge cases and leap years)
1. The **Plot the seismic graphs** section ingests the dataset and builds X / Y / Z graphs of the sensor and seismic activity.
1. The **Download a historical sensor data set** section retrieves the OpenEEW dataset using an http request node (no credentials required).
Error messages are displayed if insufficient selections have been made or if the dataset is not available.

---
<a name="example4"></a>
### Example 4 : A flow that plots historical seismic activity playback from the OpenEEW dataset

I created many Node-RED flows during my OpenEEW seismic graphing learning journey. Here is one of my beginning / simplistic iterations.
This flow reads a file from a bucket of historical sensor data from the
[OpenEEW dataset](https://s3.amazonaws.com/grillo-openeew/index.html#records/)
using an AWS cloud object storage node (you would need credentials to reproduce it).  The Node-RED flow plays the data back into a graph in a Node-RED Dashboard.  The screenshot is of a
[magnitude 7.2 earthquake in Mexico](https://blog.grillo.io/analyzing-a-magnitude-7-2-earthquake-in-mexico-using-python-6272a4ff63e3) on
[2018-02-16 23:43:00](https://s3.amazonaws.com/grillo-openeew/index.html#records/country_code=mx/device_id=006/year=2018/month=02/day=16/hour=23/40.jsonl).

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplayback-dashboard.png?raw=true "OpenEEW Dashboard")

![OpenEEW Historical Playback Flow](images/openeew-quakeplayback-flow.png?raw=true "OpenEEW flow")
---

## The OpenEEW open source project needs you

Now that you have completed these examples, you are ready to modify these example flows and your own Node-RED Dashboard to build an OpenEEW Earthquake Early Warning data visualization solution.  There are several [OpenEEW GitHub project repositories](https://github.com/openeew) that you can contribute to.

Join the cutting-edge community and build open source projects to fight back against the most pressing issues of our time. See your code deployed to help those in need.

Get free access to the [IBM Cloud](https://developer.ibm.com/dwwi/jsp/register.jsp?eventid=cfc-2020-projects)

Brush up on your cloud skills while making a real difference and get involved with these open source projects supported by the Linux Foundation.

## Additional Resources
- Another Node-RED Earthquake map example which plots USGS earthquake data can be found
[here](https://github.com/johnwalicki/Node-RED-Earthquake-Dashboard)


### Authors

- [John Walicki](https://github.com/johnwalicki)
- [Grillo](https://grillo.io)
___

Enjoy!  Give us [feedback](https://github.com/openeew/openeew-nodered/issues) if you have suggestions on how to improve this tutorial.

## License

This tutorial is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](http://www.apache.org/licenses/LICENSE-2.0.txt).
