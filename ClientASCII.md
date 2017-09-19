# Introduction #

This document describes the ASCII protocol for retrieving data from the monitor server. The strength of this ASCII interface is that it can be used from any programming language that is capable of exchanging ASCII text over a TCP socket. Java programmers should use [this](ClientJava.md) document instead.

# Establishing the Connection #

The monitor server listens for ASCII-protocol connections on a specific TCP port (8051 by default).

You can test that the server is listening by using telnet to connect to the relevant host/port. eg: `telnet myserver 8051`

# Command Summary #

The following commands are currently recognised by the server:

  * **poll:** Request the current values for a set of monitor points.
  * **poll2:** Request the current values, units and limit checks for a set of monitor points.
  * **since:** Request all updated values since a specified time.
  * **between:** Request all archival values between two nominated times.
  * **following:** Return the first record >= a specified time for a set of points.
  * **preceding:** Return the last record <= a specified time for a set of points.
  * **names:** Request the names of all points in the system.
  * **details:** Request meta-information about some points.
  * **set:** Set new values for control points.
  * **alarms:** Get current status of active/shelved priority alarms.
  * **allalarms:** Get current status of all priority alarms.
  * **ack:** Set or clear the acknowledgement for a priority alarm.
  * **shelve:** Shelve or enable a priority alarm.
  * **rsa:** Return RSA encryption exponent and modulus.
  * **rsapersist:** Return persistent RSA encryption exponent and modulus.
  * **leapseconds:** Return the dictionary of leap seconds required to calculate BAT timestamps.

Each command is outlined in further detail below.

NB: All timestamps used by the monitor server are binary atomic time (BAT) timestamps. These correspond to the number of microseconds since MJD=0. Hexadecimal notation is used, eg 0x1081fca424fe40

## poll ##

The poll command asks the server to return the most-recent values for one or more monitor points. The syntax is straightforward - but ensure that you use newline characters that correspond to the new lines in the examples below (it doesn't matter if you use \n or \n\r though).

You need to send:
```
poll
[num_points]
[pointname1]
...
[pointnameN]
```

The server will respond with:
```
[pointname1]\t[BAT_timestamp]\t[value]
...
[pointnameN]\t[BAT_timestamp]\t[value]
```

Here is a real example where we request the value for a single monitor point:
```
poll
1
site.environment.weather.Temperature
site.environment.weather.Temperature 0x1081fca424fe40 33.8
```

If the server doesn't recognise one of the monitor point names you request, it will respond with a question mark "?" in place of the line for that monitor point.

## poll2 ##

Similar to the poll command except that the response from the server also includes the data units and a boolean indicating whether the current value is within acceptable limits or not.

You need to send:
```
poll2
[num_points]
[pointname1]
...
[pointnameN]
```

The server will respond with:
```
[pointname1]\t[BAT_timestamp]\t[value]\t[units]\t[limitOK]
...
[pointnameN]\t[BAT_timestamp]\t[value]\t[units]\t[limitOK]
```

Notes:
  * The units will be displayed as "?" if the monitor point does not have units defined.
  * The limit check will return true if the current value is okay, or false if the value is currently out of range.
  * If the server doesn't recognise one of the monitor point names you request, it will respond with a question mark "?" in place of the response for that monitor point.
  * If the monitor point name is valid but no data is available "?"s will be returned for each field in the response.

Here is a real example where we request the value for a single monitor point:
```
poll2
1
mpacc.cryo.LS.20K
mpacc.cryo.LS.20K 0x10b32b0a376290 22.0 K true
```

## since ##

The since command returns all records, for a single monitor point, between the nominated time and the present. To make the request you first need to send the string "since" and then, on a new line, send the BAT timestamp for when you want the request to start and then the name of the monitor point.

If you also want to retrieve the alarm status for each record you can add the optional argument "alarms" to the query line.

For instance, the following is a valid request:
```
since
0x1065ab64134158 site.environment.weather.Temperature
```

The server will respond with:
```
[number_of_data_which_follow]
[timestamp1]\t[data1]
[timestamp2]\t[data2]
...
[timestampN]\t[dataN]
```

An example with alarm information is as follows. Each "alarm" field will either be `true` if the point was in an alarm state of `false` is there was no alarm.

```
since
0x1065ab64134158 site.environment.weather.Temperature alarms
```

The server will respond with:
```
[number_of_data_which_follow]
[timestamp1]\t[data1]\t[alarm1]
[timestamp2]\t[data2]\t[alarm2]
...
[timestampN]\t[dataN]\t[alarmN]
```


In order to prevent the server from becoming bogged down by large data requests it may impose a limit on the number of data records returned from a single call to the archive. Therefore you should process calls to 'since' in a loop which advances the request timestamp, until all data has been returned.

## between ##

The between command is similar to the since command except that it returns all records, for a single monitor point, between two nominated times (whereas since returns all data between the specified time and the present).

As with the since call, an optional "alarms" argument may be given, in which case the server will append the boolean alarm status for each returned record.

The following shows a valid request and the reply from the server:
```
between
0x10820fbd8375c0 0x10820fbfe5cfc0 site.environment.weather.Temperature
5
0x10820fbd8375c0 33.9
0x10820fbe1c0c40 33.9
0x10820fbeb4a2c0 33.9
0x10820fbf4d3940 33.9
0x10820fbfe5cfc0 33.9
```

In order to prevent the server from becoming bogged down by large data requests it may impose a limit on the number of data records returned from a single call to the archive. Therefore you should process calls to 'between' in a loop which advances the request start timestamp, until all data has been returned.

## following ##

Return a first archive records with timestamps >= specified times, for a set of monitor points. This can basically be used to step forward one step in time if you have an existing set of timestamps for a group of points.

You need to send:
```
following
[num_points]
[BAT_timestamp1] [pointname1]
...
[BAT_timestampN] [pointnameN]
```

As per the poll command, the server will respond with:
```
[pointname1]\t[BAT_timestamp]\t[value]
...
[pointnameN]\t[BAT_timestamp]\t[value]
```

Here is a real example where we request the value for a single monitor point:
```
following
1
0x1081fca4250000 site.environment.weather.Temperature
site.environment.weather.Temperature 0x1081fca424fe40 33.8
```

## preceding ##

Return a last archive records with timestamps <= specified times, for a set of monitor points. This can basically be used to step backward one step in time if you have an existing set of timestamps for a group of points.

You need to send:
```
preceding
[num_points]
[BAT_timestamp1] [pointname1]
...
[BAT_timestampN] [pointnameN]
```

As per the poll command, the server will respond with:
```
[pointname1]\t[BAT_timestamp]\t[value]
...
[pointnameN]\t[BAT_timestamp]\t[value]
```

Here is a real example where we request the value for a single monitor point:
```
preceding
1
0x1081fca4200000 site.environment.weather.Temperature
site.environment.weather.Temperature 0x1081fca424fe40 33.8
```

## names ##

The names command asks the server to print the names of all monitor points known to the system. This can be useful for identifying which monitor points are currently available (normally this wouldn't change often, but the command might prove useful to someone).

To make the request simple print the string "names" on a line by itself:
names

The server will respond with:
```
[number_of_point_names_that_follow]
[pointname1]
...
[pointnameN]
```

## details ##

The details command asks the server to send some meta-information about the specified monitor points. The format of the request is as follows:

```
details
[num_points]
[pointname1]
...
[pointnameN]
```

For each "pointname" you requested the system will respond with a line containing:
`[pointname]\t[sample_period]\t["units"]\t["description"]`
If a point name isn't recognised the server will respond with only "?" for the corresponding line of the response.

The sample period is in decimal seconds. NB, for some points the returned interval is only a rough guideline. The units and description are both delimited by "quotation marks".

Here is a complete example:
```
details
1
site.environment.weather.Temperature
site.environment.weather.Temperature 10.0 "C" "Temperature"
```

## set ##

The set command allows new values to be assigned to a specified set of points.

The username and password may be plaintext (bad idea) or both encrypted using the RSA keys available from the "rsa" or "rsapersist" calls.

You need to send:
```
set
[username]
[password]
[num_points]
[pointname1]\t[type_code]\t[value]
...
[pointnameN]\t[type_code]\t[value]
```

The type specifier must be one of the following:
| Code | Type    | Example |
|:-----|:--------|:--------|
| dbl  | Double  | 3.141 |
| flt  | Float   | 3.141 |
| int  | Integer | 7     |
| str  | String  | foo   |
| bool | Boolean | true  |
| abst | Absolute Time | 0x110ba68ede4c38 |
| relt | Relative Time | 1000000 |

The server will respond with the name of each point and OK or ERROR depending on whether the control command went through to the relevant subsystem okay:
```
[pointname1]\tOK
...
[pointnameN]\tERROR
```

If the server doesn't recognise one of the monitor point names you specified or the relevant type/value cannot be parsed, it will respond with a line beginning with a question mark "?" in place of the line for that monitor point.

## alarms ##
This returns the current list of priority points which either have an active alarm status or are shelved.

You just need to send:
```
alarms
```

The server will respond with a number, being the number of alarm point which follow, and then a tab delimited alarm status report for each of the points.

```
[number_of_alarms]
[point1]\t[priority1]\t[alarm1]\t[ack1]\t[ackby1]\t[ackat1]\t[shelved1]\t[shelvedby1]\t[shelvedat1]\t[guidance1]
...
[pointN]\t[priorityN]\t[alarmN]\t[ackN]\t[ackbyN]\t[ackatN]\t[shelvedN]\t[shelvedbyN]\t[shelvedatN]\t[guidanceN]
```

Here is an example:

```
alarms
3
site.test1	0	false	false	null	null	true	david	0x113e43a99a0358	""
site.test2	0	true	false	null	null	false	null	null	"The current value is 0.747. Please call staff."
site.test3	3	true	true	david	0x113e43a9890a89	false	null	null	"Control rod failure."
```

The fields are as follows:
|Point Name | The name of the point with which this priority alarm is associated. |
|:----------|:--------------------------------------------------------------------|
|Priority   | The priority level of this alarm. 0=Information. 1=Minot. 2=Major. 3=Severe. |
|Alarm | Indicates if the point is currently in an alarm status. Shelved points are also reported by this call but may not actively be alarming at the time. In the example above test2 and test3 are in an alarm state. |
|Acknowledged | Boolean which indicates if the alarm has been acknowledged by a network client. In the examples above the point test3 has been acknowledged by 'david'. |
|Acknowledged By | Name of the operator who acknowledged or deacknowleged the alarm. This will read 'null' if the point is still unacknowledged. |
|Acknowledged At | BAT timestamp for when the operator acknowledged or deacknowledged the alarm. This will read 'null' if the point is still unacknowledged. |
|Shelved | Boolean which indicates if the alarm has been shelved by a network client. In the example above the point test1 has been shelved by the operator named 'david'. |
|Shelved By | Name of the operator who shelved or unshelved the alarm. This will read 'null' if the point has never been shelved. |
|Shelved At | BAT timestamp for when the operator shelved or unshelved the alarm. This will read 'null' if the point has never been shelved. |
|Guidance | Guidance text to be presented to the operator to provide more information and remedial action advice. |

## allalarms ##
This returns the complete list of alarms regardless of their state. The returned data is identical to the alarms call above.

## ack ##
This is used to acknowledge or deacknowledge an alarm. The alarm will remain acknowledged or deacknowledged until the underlying alarm condition clears at which time the acknowledgement status is cleared.

To use this command you must authenticate and then specify the number of alarms that are being acknowledged and then specify the acknowledgement state for each (true for acknowledged, false for deacknowledged).

The username and password may be plaintext (bad idea) or both encrypted using the RSA keys available from the "rsa" or "rsapersist" calls.

```
ack
[username]
[password]
[num_alarms]
[pointname1]\t[ack_flag1]
...
[pointnameN]\t[ack_flagN]
```

The server will respond with the name of each point and OK, unless the authentication failed in which case the response will contain the point name and then ERROR for each point.

```
[pointname1]\tOK
...
[pointnameN]\tOK
```

If the server doesn't recognise one of the monitor point names you specified, it will respond with a line beginning with a question mark "?" in place of the line for that monitor point.

## shelve ##
The syntax for the shelve command is exactly as per ack, except that the command is called 'shelve'. The difference is that a point will remain shelved until it is later unshelved, even if the underlying alarm condition clears.

## rsa ##
Return the exponent and modulus of the servers RSA keys on separate lines. These keys will only persist as long as this particular ASCII socket session.

```
rsa
```

The server will respond with the key information.

```
3
64129427761184638068388211256513602701384755910053899609298359337859142182951474380186002214620110237483193407149019925665073596329465725365471939240275962604740323716199200201521284138192577069039566456866361990394928007375249864591359258001438102226579595169048222257001072845121797740923277078837026245439
```

## rsapersist ##
As with the "rsa" call, except the returned keys will persist for longer than the particular ASCII socket session.

## leapseconds ##
This returns a mapping between milliseconds since the 1st of January 1970 and the corresponding dUTC (leap seconds) which come into effect as various times.

The server will respond with the number of rows in the response and then each row will contain the milliseconds that a new dUTC came into effect and the corresponding dUTC, delimited with a tab.

```
numrows
[epoch 1]\t[dUTC 1]
[epoch 2]\t[dUTC 2]
...
[epoch N]\t[dUTC N]
```

An (artificially truncated) example transaction is:

```
leapseconds
4
63072000000	10
78796800000	11
94694400000	12
126230400000	13
```