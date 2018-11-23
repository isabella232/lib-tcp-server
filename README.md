# lib-tcp-server
A library for [HCL RTist](https://www.devops-community.com/realtime-software-tooling-rtist.html) which allows an RTist application to communicate over TCP with another application.
Note: This library requires RTist 10.3 2018.48 or later.

## Usage
This library contains a capsule for letting an external application communicate with an RTist application over TCP. It's possible to implement a custom handling of incoming messages, but the default implementation supports messages on a certain JSON format. It allows external applications to send events on the ports of your capsule that inherits from `TCPServer`. Here is an example:

`
{ "command": "sendEvent", "port" : "trafficLight", "event" : "test_int", "data" : "int 15" }
`

For now the syntax of the "data" follows the standard ASCII format used by RTist (for example when tracing using the model debugger).

The response is also on JSON format. Here is an example of an error response:

`
"{"status":"error", msg:"Failed to parse JSON: JSON Exception: error: 1: unexpected byte, 'c'"}"
`
