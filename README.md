# lib-tcp-server
A library for [HCL RTist](https://www.devops-community.com/realtime-software-tooling-rtist.html) which allows an RTist application to communicate over TCP with other applications.
Note: This library requires RTist 10.3 2018.48 or later.

## Usage
<img src="https://github.com/hcl-pnp-rtist/lib-tcp-server/blob/master/img/usage.jpg" width="805" height="358">

This library contains a capsule `TCPServer` which allows external applications to communicate with an RTist application over TCP. Create a capsule in your model that inherits from `TCPServer`, and then create a capsule part typed by your capsule. This capsule part (referrred to as the "TCPServer capsule part" below) acts as the connection point to the remote application(s) that your application now can communicate with.
It's possible to implement a custom handling of incoming messages, but the default implementation supports messages on a certain JSON format. It allows external applications to, for example, send events on the ports of your capsule that inherits from `TCPServer`. Here is an example:

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`

To implement your own custom handling of received messages, override the operation `TCPServer::handleReceivedMessageCustom()` and set the config property `defaultHandlingOfReceivedMessages` to false.

## Build the Library
The library uses the [POCO C++ libraries](https://pocoproject.org) and you must therefore first clone and build the [POCO GitHub repository](https://github.com/pocoproject/poco) for your particular platform.

Add the TC `tcpServerLib.tcjs` as a prerequisite of the executable TC for your RTist application. Also specify the location of the POCO library root folder by setting the 'pocoLoc' property of your TC (using the Code tab of the TC editor). For example:

`
tc.pocoLoc = 'D:\\github\\poco';
`

Also modify your executable TC to add the paths to POCO include files. Use the property you just added:

`
tc.inclusionPaths = [
    tc.pocoLoc + '\\Net\\include',
    tc.pocoLoc + '\\JSON\\include',
    tc.pocoLoc + '\\Foundation\\include'
];
`

Also tell the linker where to find the built POCO libraries. For example, for Visual Studio 64 bit set the following:

`
tc.linkArguments = '/MACHINE:X64 /LIBPATH:"' + tc.pocoLoc + '\\lib64"';
`

Finally, if you have built POCO as shared libraries remember to set the PATH variable (on Windows) or LD\_LIBRARY\_PATH (on Unix) to include the folder where they are located. For 64 bit builds the folder is called 'bin64' and is in the POCO root folder.

## Configuration Properties
Configuration properties are defined as static attributes of the `TCPServer_Config` class. If you don't want to change their default values in the library, you have to programmatically override the attribute values you want to change. You can do this by overriding the operation `init()` in your capsule that inherits from `TCPServer`. For example:

`
config.remotePort = 2235;
`

The following configuration properties are available:

- **defaultHandlingOfReceivedMessages**:boolean 
If true, all incoming TCP messages are assumed to be string-encoded JSON objects according to the format described below. Set to false if you wish to implement your own custom handling of incoming messages.
- **maxWaitForReply**:integer
The max number of milliseconds to wait for the RTist application to reply to a request for sending or invoking an event to it. If the TCPServer doesn't get a confirmation from RTist within the specified time limit, it will assume that the RTist application failed to receive the event and return an error message. 
- **port**:integer
The TCP port in the RTist application used for incoming messages.
- **remoteHost**:string
The name or IP address of the machine where the remote application (that will receive outgoing messages) runs.
- **remotePort**:integer
The TCP port on which the remote application (that will receive outgoing messages) listens.


## JSON Format of Incoming Messages
The default implementation (in `TCPServer::handleReceivedMessageDefault()`) expects incoming TCP messages to be a string encoded JSON object. The object must have the property `command` set to one of the commands listed below. Which other properties on the JSON object that are expected depends on the command.

### sendEvent
Send an event into the application through a port on the TCPServer capsule part.
Example:  

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`

- **port**:string *[mandatory]*
The name of a port on the TCPServer capsule part through which to send the event.
- **event**:string *[mandatory]*
The name of an event defined in the protocol that types the port specified above. The event should be an "outEvent" if the port is not conjugated. If it is conjugated the event should be an "inEvent".
- **data**:string *[optional]*
The RTist ASCII encoding of the data object to send with the event. This encoding is on the same format as you for example see if tracing the event during a model debug session.
- **portIndex**:integer *[optional]*
If the port is replicated (i.e. its multiplicity is > 1) you can specify the index of the port instance you want to send the message to. Omitting the portIndex on a replicated port means that the event will be broadcasted through all port instances.
- **priority**:string *[optional]*
The priority at which the event should be sent. The property defaults to "General". Valid priorities are (from highest to lowest) "Panic", "High", "General", "Low" and "Background". 

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
The name of a port on the TCPServer capsule part through which to invoke the event.
- **event**:string *[mandatory]*
The name of an event defined in the protocol that types the port specified above. The event should be an "outEvent" if the port is not conjugated. If it is conjugated the event should be an "inEvent".
- **data**:string *[optional]*
The RTist ASCII encoding of the data object to send with the event. This encoding is on the same format as you for example see if tracing the event during a model debug session.
- **portIndex**:integer *[optional]*
If the port is replicated (i.e. its multiplicity is > 1) you can specify the index of the port instance you want to send the message to. Omitting the portIndex on a replicated port means that the event will be broadcasted through all port instances
. 

The response is a JSON object with the following properties:

- **status**:string 
'ok' if the command was successful, otherwise 'error'
- **msg**:string
A message clarifying the status (especially if the status is 'error').
- **result**:array
An array of JSON objects representing the reply messages. There will be one object for each reply. The objects have the following properties.
  - **event**:string
  Name of the reply event.
  - **type**:string
  Data type of the reply event.
  - **data**:any *[optional]*
  The RTist JSON encoding of the data object attached to the reply event. The type of this property is determined by the "type" property and can either be a string, a boolean, an integer, an array or an object.
  - **isValid**:boolean *[optional]*
  This property is only present (and set to false) if the JSON object represents an invalid reply event. This for example happens if the receiver did not make an explicit reply, even if the reply event contains data. No other properties will be present for an invalid reply event.

## JSON Format of Outgoing Messages
All messages that are not handled by the TCPServer capsule part are considered to be outgoing messages and will be sent to a remote application. Note that the remote application to which outgoing messages are sent does not have to be the same remote application that sends incoming messages to the RTist application.

Outgoing messages are string encoded JSON objects on the same form as the 'sendEvent' objects described above. This makes it possible to wire together multiple RTist applications to build a distributed system of communicating applications. The output of one executable can be consumed by another executable, and the applications become logically connected through the names of the TCPServer capsule ports.

- **event**:string
Name of the sent event.
- **type**:string
Data type of the sent event.
- **data**:any *[optional]*
The RTist JSON encoding of the data object attached to the sent event. The type of this property is determined by the "type" property and can either be a string, a boolean, an integer, an array or an object.
- **port**:string
The name of a port on the TCPServer capsule part on which the event arrived.
- **portIndex**:integer
The index of the port instance on which the event arrived. If the port is not replicated (i.e. its multiplicity is 1) then this property is always 1.
- **priority**:string
The priority at which the event is sent. The property defaults to "General". Possible priorities are (from highest to lowest) "Panic", "High", "General", "Low" and "Background". 
