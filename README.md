# lib-tcp-server
A library for HCL RTist which allows an RTist application to communicate over TCP with another application.

## Usage
This library contains a capsule for letting an external application communicate with an RTist application over TCP. It's possible to implement a custom handling of incoming messages, but the default implementation supports messages on a certain JSON format. It allows external applications to send events on the ports of your capsule that inherits from `TCPServer`. Here is an example:

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`
