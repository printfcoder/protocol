# Micro Communication Protocol (MUCP)

This repository provides a home for the development and specification for the **micro communication protocol** (mucp). 

## Overview

The micro communication protocol is a simple design specfication for how micro services sends and receives messages over any transport. 
Our intention is to define an open protocol for service to service communication on top of any system. This also provides us the ability 
to simplify the creation of clients and servers which accept micro based requests. 

Ideally client should be implemented using the micro proxy http interface to standardise creation.

## Protocol

The protocol is incredibly simple. We define a set of headers and an encoded message body for bidirection request/response streaming and 
asynchronous messaging.

Where the transport or broker accepts headers (such as http) the headers will be encoded in its headers. Otherwise 
the entire header and message will be encoded in an envelope. Our preference is to use protobuf but the protocol 
should scan for a starting json delimiter `{` to know whether to decode to json.

### Request/Response

The request/response are identical in format. The request includes headers id, service and endpoint. The response echoes these 
back the same headers but includes the response body.

An example request.

```
{
	Header: {
		"Micro-Id": "d02d5da0-14dc-11e9-ab14-d663bd873d93",
		"Micro-Service": "greeter",
		"Micro-Endpoint": "Say.Hello",
		"Content-Type": "application/protobuf",
	}
	Body: []byte(...)
}
```

In the event of an error we return it as a header (may change to body).

```
{
	Header: {
		"Micro-Id": "d02d5da0-14dc-11e9-ab14-d663bd873d93",
		"Micro-Service": "greeter",
		"Micro-Endpoint": "Say.Hello",
		"Micro-Error": {"id":"greeter.Say.Hello","code":500,"detail":"Failed greeting","status":"Internal Server Error"},
	}
}
```

## Publication

Messages can be published asynchronously via the protocol. These are sent to a topic which may have multiple subscribers. The sender 
does not need to know where the subscribers live or if there are any at the time. Ideally the proxy or accepter of 
the message should save the message to an inbox for a period of time if no subscribers are present so they 
can be retrieved later.

An example message.

```
{
	Header: {
		"Micro-Id": "d02d5da0-14dc-11e9-ab14-d663bd873d93",
		"Micro-Topic": "events",
		"Content-Type": "application/protobuf",
	}
	Body: []byte(...)
}
```

In the event you want to subscribe to a topic you must specify a queue.

```
{
        Header: {
                "Micro-Id": "d02d5da0-14dc-11e9-ab14-d663bd873d93",
                "Micro-Topic": "events",
                "Micro-Queue": "customer",
        }
}
```

### Optional

We are looking at optional extra headers for routing

- `Micro-Protocol` to specify the protocol for the endpoint. 
- `Micro-Stream` to segregate streams on the same connection.
- `Micro-Channel` or `Micro-Queue` to specify a specific queue to segregate by.
