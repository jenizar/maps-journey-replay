# maps-journey-replay
Javascript Library that loads a journey JSON object (coordinates + timestamp) and creates a replay on a map.

![alt text](https://github.com/jenizar/maps-journey-replay/tree/master/demo/images)

# Requirement
Google Map API Key

# Installation
1. /var/www/html/$ git clone : https://github.com/jenizar/maps-journey-replay.git 
2. $ cd maps-journey-replay/demo
3. gedit journey-replay.html
4. insert your Google Map API Key
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;libraries=geometry&amp;callback=initMap"></script>

Browse http://localhost/maps-journey-replay/demo/journey-replay.html

## Demo

[Link to Demo](https://sebastianvirlan.github.io/maps-journey-replay/)

## Journey JSON format

```JSON
[
  {
    "lat": 53.94632335995299,
    "lng": -1.3704424708440581,
    "timestamp": 1486291045000
  },
  {
    "lat": 53.94624075051081,
    "lng": -1.3701714873316115,
    "timestamp": 1486291052000
  },
  .....
 ]
```

## Initializing the library example:

```javascript
    var mapJourneyReplay = new journeyReplay({
        type: journeyReplay.REPLAY_TYPE.DYNAMIC_MARKER_AND_POLY_LINES_COLORED_BY_SPEED,
        logger: console.log,
        gradientGenerator: function (options) {
            return new GradientGenerator({colours: options.speedColours, steps: options.steps}).hex()
        },
        fetchMapPoints: {
            url: 'http://192.168.64.2/timeline.json',
            dataMap: function (data) {
                // If data is not coming in the required format, the dataMap will be used for transforming the data
                var locations = data.locations.map(function(val) {
                    return {
                        lat: val.latitudeE7 * (10 ** -7),
                        lng: val.longitudeE7 * (10 ** -7),
                        timestamp: parseInt(val.timestampMs)
                    }
                }).sort(function(a, b) { return a.timestamp - b.timestamp })

                return locations
            }
        },
        mapProvider: googleMaps,
        marker: {
            icon: 'images/bus.svg',
            anchor: [39, 60],
            size: [100, 100],
            scaledSize: [100, 75],
            onFrame: function (currentFrame, speed, distance, animationHandler) {
                // do stuff
            },
            onLocation: function (currentLocation) {
                // do stuff
            }
        },
        speedColours : ["#00ff00", "#ffff00", "#ff0000"],
        colourSteps : 100
    });

    mapJourneyReplay.marker.onFrame(function (currentFrame, speed, distance, animationHandler) {
        // do stuff
    });

    mapJourneyReplay.marker.onLocation(function (currentLocation) {
        // do stuff
    });

    journeyReplay.init();
```

## Options

|       Key      |                         Default Value                         |            Value Type           | Description                                                                                                                                                                                                                                                                              |
|:--------------:|:-------------------------------------------------------------:|:-------------------------------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|      type      |                        `STATIC_MARKER`                        |             `string`            | Specifies how to replay the journey. There are 5 replay types:                                                                                                                                                                                                                           |
|                |                                                               |                                 | - STATIC_MARKER - moves the marker from location to location without any animation                                                                                                                                                                                                       |
|                |                                                               |                                 | - STATIC_MARKER_AND_POLY_LINES - jumps the marker from location to location without any animation and draws a polyline between locations                                                                                                                                                 |
|                |                                                               |                                 | - DYNAMIC_MARKER - moves the marker from location to location with animation                                                                                                                                                                                                             |
|                |                                                               |                                 | - DYNAMIC_MARKER_AND_POLY_LINES - moves the marker from location to location with animation and draws a polyline between locations                                                                                                                                                       |
|                |                                                               |                                 | - DYNAMIC_MARKER_AND_POLY_LINES_COLORED_BY_SPEED - moves the marker from location to location with animation and draws a colorized polyline between locations based on the speed (calculated from the timestamp)                                                                         |
|     logger     |                         `console.log`                         | `function (message: string) {}` | Logs to the output informations useful for debugging. By default it uses console.log, other options are: - null - customLogger(message: string)                                                                                                                                          |
|      domID     |                             `map`                             |             `string`            | The id of the DOM element where the map is loaded.                                                                                                                                                                                                                                       |
| fetchMapPoints |  `{   url: null,   onDone: function () {},   dataMap: null}`  |     `object: FetchMapPoints`    | fetchMapPoints is an object that has 3 properties:  - url that is a string - onDone which is called after the JSON has been loaded - dataMap for transforming the JSON received from the server so that the library can understand it                                                    |
|   mapProvider  |                          `googleMaps`                         |         `function: Maps`        | mapProvider takes a constructor function that must implement a list of methods See `Maps interface` to see all the methods that needs implemented. By using this interface maps from other providers can be implemented as well. By default the implemented interface is for googleMaps. |
|     marker     | `{   onFrame: function () {},   onLocation: function () {} }` |         `object Marker`         | The marker key contains 2 callbacks: - onFrame, called for each frame of the animation - onLocation, called for each location from the JSON file                                                                                                                                         |
|   speedColors  |              `["#00ff00", "#ffff00", "#ff0000"]`              |            `string[]`           | Array with hex colours that represents the polyline colorisation by speed. The first element in the array is the lowest speed colour and the last is the highest speed colour. This property must be used with the colourSteps option.                                                   |
|   colorSteps   |                             `100`                             |              `int`              | Defines in how many steps you can get the highest speed colour from the lowest. Each step represents 1km, so after 100km will be used the highest speed colour.                                                                                                                          |

## Default Options Object

```javascript
var defaults = {
    logger          : console.log,
    type            : REPLAY_TYPE.STATIC_MARKER,
    domID           : 'map',
    mapProvider     : null,
    colorSteps      : 100,
    speedColors     : ["#00ff00", "#ffff00", "#ff0000"],
    fetchMapPoints: {
        url: null,
        onDone: function () {},
        dataMap: null
    },
    marker: {
        icon: '../images/bus.svg',
        anchor: [39, 60],
        size: [100, 100],
        scaledSize: [100, 75],
        onFrame: function () {},
        onLocation: function () {}
    }
};
```

## Maps Interface

```javascript
function Maps () {
  this.initMap = function (options) {}
  this.onMapIdle = function (callback) {}
  this.setCenter = function (latLng) {}
  this.marker = function (icon, location) {
      this.changeIcon = function (iconPath) {}
      this.rotate = function (rotationAngle) {}
      this.remove = function () {}
      this.getLocation = function () {}
      this.setPosition = function (newLatLng) {}
      this.animateTo = function (newLocation, options) {}
  }
  this.polyLine = function (options) {
      this.remove = function () {}
  }
  this.resetMap = function () {}
  this.removeMarkers = function () {}
  this.removePolylines = function () {}
}
```

## Objects

```javascript
var fetchMapPoints = {
    url: 'string',
    onDone: function () {}
}

var marker = {
    onFrame: function () {},
    onLocation: function () {}
}
```
