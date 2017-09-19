# Introduction #

This document describes the RESTlet RESTful interface which allows MoniCA data to be queries via standard HTTP queries.

# Obtaining Metadata #

## Point Names ##

The list of all points which exist on the server can be obtained with a GET/POST query to the **/names/** URL, eg:

```
http://bullawa:8111/names
```

The response takes the following format:

```
{"status":"ok", "errorMsg":"", "monitoringPointNames":["home.weather.wind_speed","home.weather.wind_dir"]}
```

## Point Descriptions ##

Point description metadata can be obtained in a number of ways. To obtain the metadata for all points which are defined on the server you can perform a GET/POST request to the **/descriptions/** URL.

```
http://bullawa:8111/descriptions
```

This operation may be expensive on a server which has thousands of points defined, in which case selectively obtaining point descriptions as required may be preferable.

To request data for a single point via the GET/POST method you can query the **/description/** URL with a single point name argument:

```
http://bullawa:8111/description/home.weather.wind_speed
```

To request data for multiple points via the GET/POST (form) method you can query the **/description/** URL with semi-colon separated list of point names:
```
http://bullawa:8111/descriptions?points=home.weather.wind_speed;home.weather.wind_dir
```

To query multiple point definitions via a POST(JSON) interface, you need to pass a json object to the **/descriptions/** URL with the following format:

```
{"points": ["home.weather.wind_speed", "home.weather.wind_dir"]}
```

In all cases the response will take the following format:

```
{"status":"ok", "errorMsg":"", "monitoringPointDescriptions": [{"name":"home.weather.wind_speed","desc":"Wind Speed","units":"km/h","period":2.5},{"name":"home.weather.wind_dir","desc":"Wind Direction","units":"deg","period":2.5}]}
```

The _status_ field will be 'ok' unless there was an error in which case it will say 'fail'. If status is 'fail', you might a get a reason why it failed in errorMsg. The _descriptions_ array contains the actual descriptions, each of which may contain the following fields:

  * **name:** The name of the point this metadata belongs to.
  * **desc:** If the point has a description defined then this field will exist and contain that description.
  * **units:** If the point has units defined then this field will exist and contain the units.
  * **period:** If the point has an update period defined then this field will exist and specify the period in floating point seconds.

# Obtaining Data #

A single datum may contain the following fields:
  * **name:** The name of the point that this datum pertains to.
  * **ts:** The timestamp of the data.
  * **value:** The data value, if this field is omitted then a data value of _null_ may be assumed.
  * **alarm:** Whether the point was in an alarm state. If this field is omitted then the alarm state can be assumed to be _false_.

## POST (JSON) Interface ##

This accepts a POST query in JSON format which can query one or multiple points in a single operation. The result is a JSON object containing the requested data.

The query form action URL must be **/points/**. eg `http://bullawa:8111/points`

```
{"type": "get", "points": ["home.weather.wind_speed", "home.weather.wind_dir"]}'
```


The result will contain the latest data for each point. If any of the point names were not known then there will be no data for that point. I all of the point names were not known then the status will be "fail".
```
{"status":"ok", "pointData":[{"pointName":"home.weather.wind_speed","time":"2012-04-13 03:49:51.926","value":17.71,"errorState":true}, {"pointName":"home.weather.wind_dir","time":"2012-04-13 03:49:51.926","value":163.0}]}
```

### Between ###

This operation returns data from the archive in between the specified start and end time. Although the point names can be specified as an array, only a single point may be specified for this query:

```
{"type":"between","start":"2012-04-13 03:34:30","end":"2012-04-13 03:35:00","points":["home.weather.wind_dir"]}
```

The response should look like the example below. Please note that in the interests of efficiency the name of the point is excluded from the individual results of this query:
```
{"status":"ok", "pointData":[{"time":"2012-04-13 03:34:31.044","value":203.0}, {"time":"2012-04-13 03:34:33.622","value":186.0}, {"time":"2012-04-13 03:34:36.201","value":179.0}, {"time":"2012-04-13 03:34:38.777","value":194.0}, {"time":"2012-04-13 03:34:41.353","value":130.0}, {"time":"2012-04-13 03:34:43.929","value":189.0}, {"time":"2012-04-13 03:34:46.506","value":183.0}, {"time":"2012-04-13 03:34:49.087","value":175.0}, {"time":"2012-04-13 03:34:51.665","value":179.0}, {"time":"2012-04-13 03:34:54.242","value":187.0}, {"time":"2012-04-13 03:34:56.818","value":206.0}, {"time":"2012-04-13 03:34:59.394","value":179.0}]}
```

### After ###

Queries the first values available for a set of points which are after or equals the specified time:

```
{"type":"after","time":"2012-04-13 03:59:00","points":["home.weather.wind_speed","home.weather.wind_dir"]}
```

Example result:
```
{"status":"ok", "pointData":[{"pointName":"home.weather.wind_speed","time":"2012-04-13 03:59:16.265","value":9.66,"errorState":true}, {"pointName":"home.weather.wind_dir","time":"2012-04-13 03:59:16.265","value":135.0}]}
```

### Before ###

As with the "after" query above, except the query type must be specified as "before".

## GET and Post (form) interface ##

This accepts GET query parameters for a single point.

### Get ###

To get the latest value for a single point, you can use a query in the following format:

```
http://bullawa:8111/point/home.weather.wind_dir
```

For multiple points:
```
http://bullawa:8111/points?type=get&points=home.weather.wind_speed;home.weather.wind_dir
```

The server will respond with a JSON result like:
```
{"status":"ok", "pointData":[{"pointName":"home.weather.wind_dir","time":"2012-04-13 03:31:39.600","value":200.0}]}
```



### Between ###

To query all of the data for a single point between two nominated times you can use the following:

```
http://bullawa:8111/point/home.weather.wind_dir?type=between&start=2012-04-13%2003:34:30&end=2012-04-13%2003:35:00
```

```
http://bullawa:8111/points?points=home.weather.wind_dir;home.weather.wind_speed&type=between&start=2012-04-13%2003:34:30&end=2012-04-13%2003:35:00
```

The response should look like the example below. Please note that in the interests of efficiency the name of the point is excluded from the individual results of this query:
```
{"status":"ok", "pointData":[{"time":"2012-04-13 03:34:31.044","value":203.0}, {"time":"2012-04-13 03:34:33.622","value":186.0}, {"time":"2012-04-13 03:34:36.201","value":179.0}, {"time":"2012-04-13 03:34:38.777","value":194.0}, {"time":"2012-04-13 03:34:41.353","value":130.0}, {"time":"2012-04-13 03:34:43.929","value":189.0}, {"time":"2012-04-13 03:34:46.506","value":183.0}, {"time":"2012-04-13 03:34:49.087","value":175.0}, {"time":"2012-04-13 03:34:51.665","value":179.0}, {"time":"2012-04-13 03:34:54.242","value":187.0}, {"time":"2012-04-13 03:34:56.818","value":206.0}, {"time":"2012-04-13 03:34:59.394","value":179.0}]}
```

### After ###

To query the first value in the archive for a particular point which is after or equals a specified time you can use the following syntax:

```
http://bullawa:8111/point/home.weather.wind_dir?after=2012-04-13%2003:34:30
```

```
http://bullawa:8111/points?points=home.weather.wind_dir;home.weather.wind_speed&type=after&time=2012-04-13%2003:34:30
```

A single point should be returned:
```
{"status":"ok", "pointData":[{"pointName":"home.weather.wind_dir","time":"2012-04-13 03:42:27.279","value":172.0}]}
```

### Before ###

As with the "after" query above, except the query type must be specified as "before".