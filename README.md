# lib-tcp-server
A library for HCL RTist which allows an RTist application to communicate over TCP with another application.

## Usage
This library contains a capsule `TCPServer` which allows an external application communicate with an RTist application over TCP. Create a capsule in your model that inherits from `TCPServer`, and then create a capsule part typed by your capsule. This capsule part (referrred to as the "TCPServer capsule part" below) acts as the connection point to the remote application that your application now can communicate with.
It's possible to implement a custom handling of incoming messages, but the default implementation supports messages on a certain JSON format. It allows external applications to send events on the ports of your capsule that inherits from `TCPServer`. Here is an example:

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`

To implement your own custom handling of received messages, override the operation `TCPServer::handleReceivedMessageCustom()` and set the config property `defaultHandlingOfReceivedMessages` to false.

## JSON Format
The default implementation (in `TCPServer::handleReceivedMessageDefault()`) expects incoming TCP messages to be a JSON object. The object must have the property `command` set to one of the commands listed below. Which other properties on the JSON object that are expected depends on the command.

### sendEvent
Send an event into the application through a port on the TCPServer capsule part.
Example:  

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`

- **port**:string *[mandatory]*
The name of a port on the TCPServer capsule part through which to send the event.
- **event**:string *[mandatory]*
The name of an event defined in the protocol that types the port specified above. The event should be an "outEvent" if the port is not conjugated. If it is conjugared the event should be an "inEvent".
- **data**:string *[optional]*
The RTist ASCII encoding of the data object to send with the event. This encoding is on the same format as you for example see if tracing the event during a model debug session.
- **portIndex**:int *[optional]*
If the port is replicated (i.e. its multiplicity is > 1) you can specify the index of the port instance you want to send the message to. Omitting the portIndex on a replicated port means that the event will be broadcasted through all port instances
. 

The response is a JSON object with the following properties:

- **status**:string 
'ok' if the command was successful, otherwise 'error'
- **msg**:string
A message clarifying the status (especially if the status is 'error').


### invokeEvent
Invoke an event into the application through a port on the TCPServer capsule part. The reply message(s) are returned.
Example:  

`
{ "command": "invokeEvent", "port" : "tester", "portIndex" : 6, "event" : "inv", "data" : "int 12" }
`

- **port**:string *[mandatory]*
The name of a port on the TCPServer capsule part through which to send the event.
- **event**:string *[mandatory]*
The name of an event defined in the protocol that types the port specified above. The event should be an "outEvent" if the port is not conjugated. If it is conjugared the event should be an "inEvent".
- **data**:string *[optional]*
The RTist ASCII encoding of the data object to send with the event. This encoding is on the same format as you for example see if tracing the event during a model debug session.
- **portIndex**:int *[optional]*
If the port is replicated (i.e. its multiplicity is > 1) you can specify the index of the port instance you want to send the message to. Omitting the portIndex on a replicated port means that the event will be broadcasted through all port instances
. 

The response is a JSON object with the following properties:

- **status**:string 
'ok' if the command was successful, otherwise 'error'
- **msg**:string
A message clarifying the status (especially if the status is 'error').
- **result**:array
An array of JSON objects representing the reply messages. There will be one object for each reply. The objects have with the following properties.
  - **_event**:string
  Name of the reply event.
  - **_type**:string
  Data type of the reply event.
  - **_data**:any
  The RTist JSON encoding of the data object sent with the reply event. The type is determined by the "_type" property and can either be a string, a boolean or an integer.