# Introduction #

MoniCA is able to provide a service whereby clients can subscribe to updates for a nominated set of points and the server will publish updates for those points to an IceStorm topic. This service is disabled by default because it depends on having access to an IceGrid locator service and IceStorm server, however in environments where those are available then the service can be easily configured.

# Making Subscriptions #

The server creates a topic on which it will listen for new subscription requests. Clients make new subscriptions by populating the appropriate data structure (defined below) and publishing it on the server's subscription topic.

As part of the subscription the procedure the client is expected to create a new topic, on which the server will publish the actual data updates, and the name of this topic is provided to the server along with the list of points to be monitored. The Ice data structures and interface methods used to create a new subscription are shown below:

```
//Structure for subscribing to updates via an IceStorm topic
struct PubSubRequest {
  string topicname;
  stringarray pointnames;
};
        
interface PubSubControl {
  ////////////
  //Subscribe to updates via an IceStorm topic.
  void subscribe(PubSubRequest req);
          
  ////////////
  //Cancel the subscriptions through the given topic
  void unsubscribe(string topicname);
          
  ////////////
  //Notify the server that the specified topic is still active.
  void keepalive(string topicname);
};
```

The client is required to send a keep-alive message with the name of it's topic periodically (faster than once per minute by default). This ensures that the server can remove topics abandoned by clients which are no longer active to prevent build-up of unused topics on the IceStorm server.

When the client no longer wishes to receive updates for the nominated set of points then it should call the `unsubscribe` method, giving the name of the defunct topic.

# Receiving Updates #

In order to receive updates the client must implement the `PubSubClient` interface and use this to subscribe to the topic it created. This interface is shown below:

```
interface PubSubClient {
  ////////////
  //Receive new updates for one or more points
  void updateData(pointdataset newdata);
};
```

The server will publish new point data objects to the topic whenever one or more of the listened-to points updates.

# Server Configuration #

In order to enable the service a certain set of keys must be defined in the `config/monitor-config.txt` configuration file. The keys are as follows:

  * **PubSubEnabled** A key which instructs MoniCA whether it should attempt to start this service. Must be set to `true` to enable it.
  * **PubSubLocatorHost** The host name for the location of the IceGrid registry/locator service.
  * **PubSubLocatorPort** The port number for the registry.
  * **PubSubTopic** The name of the topic on which the server should listen for new subscription requests and keep-alive traffic. This topic name is essentially arbitrary but must be coordinated with clients who will need to use the topic.

An example configuration block is shown below:

```
#Settings for IceStorm pub/sub service
PubSubEnabled     true
PubSubLocatorHost localhost
PubSubLocatorPort 4061
PubSubTopic       MoniCA.PubSubControl
```

# Examples #

The open-monica class `atnf.atoms.mon.comms.PubSubClientI` provides an implementation of a client, which can be used by Java applications or run interactively.

When run interactively, the program requires the following arguments:

```
USAGE: Requires the following arguments:
	Host name for locator service.
	Port number for locator service.
	Name of control topic for making subscriptions.
	Name of point to subscribe to.
```

An example of invoking the client would be:

```
java -cp open-monica.jar atnf.atoms.mon.comms.PubSubClientI localhost 4061 MoniCA.PubSubControl site.power.UnderVolts
```

This will continue to publish data to STDOUT until it is terminated with a control-c.