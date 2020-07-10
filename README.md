# Node-RED Open Earthquake Early Warning Alert Map
Build a Earthquake Early Warning Alert Map Node-RED Dashboard using OpenEEW data feed

These example flows and Node-RED Dashboard might be useful as part of an Earthquake Early Warning system.

Another Node-RED Earthquake map example which plots USGS earthquake data can be found
[here](https://github.com/johnwalicki/Node-RED-Earthquake-Dashboard)

### Prerequistes

- [Install Node-RED](https://nodered.org/docs/getting-started/) on your system or in the cloud
- [Add the following nodes](https://nodered.org/docs/user-guide/runtime/adding-nodes) to your Node-RED palette
  - [node-red-dashboard](https://flows.nodered.org/node/node-red-dashboard)
  - [node-red-contrib-web-worldmap](https://flows.nodered.org/node/node-red-contrib-web-worldmap)
  - [node-red-node-twilio](https://flows.nodered.org/node/node-red-node-twilio)
  - [node-red-node-ui-table](https://flows.nodered.org/node/node-red-node-ui-table)
  - [node-red-node-aws](https://flows.nodered.org/node/node-red-node-aws)

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
---
### A flow that displays Earthquake Early Warning Sensor Alerts on a map

![OpenEEW Alert Dashboard](images/openeew-quakemap-v1-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-quakemap-v2.json">Node-RED flow for OpenEEW Alerts</strong></a>
</p>

![OpenEEW Sensor Plot flow](images/openeew-quakemap-v2-flow.png?raw=true "OpenEEW flow")
---
### A flow that plots (near) real time Seismic activity sensor graphs in a chart

![OpenEEW Sensor Dashboard](images/openeew-sensorplot-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-sensorplot.json">Node-RED flow for OpenEEW Sensor graphs</strong></a>
</p>

![OpenEEW Sensor flow](images/openeew-sensorplot-flow.png?raw=true "OpenEEW flow")

---
### Select a Sensor and time range, plot the historical seismic activity from that OpenEEW dataset

This flow displays a Node-RED dashboard which presents the investigator / seismologist with a calendar,
time interval options and a list of OpenEEW sensors. The investigator selects an interesting sensor
and time period to study and queries the OpenEEW dataset. The flow then plots the historical sensor
data in a set of graphs. For this flow to work correctly, you will need credentials to the AWS bucket.

The flow constructs a cloud object storage filename for the selected sensor and time interval and retrieves the historical sensor data from the
[OpenEEW dataset](https://s3.amazonaws.com/grillo-openeew/index.html#records/)
and plays it back into a graph in a Node-RED Dashboard.  

The "Next" button in the Historical Quake Playback Node-RED dashboard is incredibly insightful. The focus is often on the big earthquakes, and their specific timestamp / dataset.  The "Next" button lets the investigator explore the numerous aftershocks.  Start with the Mexico 7.2 magnitude earthquake on Feb 16 2018 23:35 and its data set and then click the Next button to watch (in 5 minute increments) the aftershocks that occurred over the next several hours.

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplaybackv4-dashboard.png?raw=true "OpenEEW Dashboard")

Here is another screenshot of a
magnitude 4.1 earthquake in Puerto Rico on
[2020-06-01 12:05:51](https://s3.amazonaws.com/grillo-openeew/index.html#records/country_code=pr/device_id=3ef3d787af85/year=2020/month=06/day=01/hour=12/05.jsonl).

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplaybackv3-dashboard.png?raw=true "OpenEEW Dashboard")
<p align="center">
  <strong>Get the Code: <a href="flows/openeew-quakemap-v4.json">Node-RED flow for OpenEEW Historical Playback</strong></a>
</p>

![OpenEEW Historical Playback Flow](images/openeew-quakeplaybackv4-flow.png?raw=true "OpenEEW flow")

---
### A flow that plots historical seismic activity playback from the OpenEEW dataset

This flow reads a file from a bucket of historical sensor data from the
[OpenEEW dataset](https://s3.amazonaws.com/grillo-openeew/index.html#records/)
and plays it back into a graph in a Node-RED Dashboard.  The screenshot is of a
[magnitude 7.2 earthquake in Mexico](https://blog.grillo.io/analyzing-a-magnitude-7-2-earthquake-in-mexico-using-python-6272a4ff63e3) on
[2018-02-16 23:43:00](https://s3.amazonaws.com/grillo-openeew/index.html#records/country_code=mx/device_id=006/year=2018/month=02/day=16/hour=23/40.jsonl).
For this flow to work correctly, you will need credentials to the AWS bucket.

![OpenEEW Historical Playback Dashboard](images/openeew-quakeplayback-dashboard.png?raw=true "OpenEEW Dashboard")

![OpenEEW Historical Playback Flow](images/openeew-quakeplayback-flow.png?raw=true "OpenEEW flow")
---

### Authors

- [John Walicki](https://github.com/johnwalicki)
- [Grillo](https://grillo.io)
___

Enjoy!  Give us [feedback](https://github.com/grillo/openeew-nodered/issues) if you have suggestions on how to improve this tutorial.

## License

This tutorial is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](http://www.apache.org/licenses/LICENSE-2.0.txt).
