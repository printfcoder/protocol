# The Micro Protocol

This repository provides a home for the development and specification for the **micro protocol**. 

## Overview

The micro protocol is a simple design specfication for how micro sends and receives messages over any transport or broker. 
Our intention is to simplify the creation of clients and servers which accept micro based requests. Ideally this 
should be proxied through a micro proxy to simplify the implementation requirements to just HTTP.

## Protocol

The protocol is incredibly simple. We define a set of headers and an encoded message body.

Where the transport or broker accepts headers (such as http) the headers will be encoded in its headers. Otherwise 
the entire header and message will be encoded in an envelope. Our preference is to use protobuf but the protocol 
should scan for a starting json delimiter `{` to know whether to decode to json.

### Request/Response

The request/response are identical in format. The request includes headers id, service and method. The response echoes these 
back the same headers but includes the response body.

An example request.

```
{
	Header: {
		"X-Micro-Id": "d02d5da0-14dc-11e9-ab14-d663bd873d93",
		"X-Micro-Service": "greeter",
		"X-Micro-Method": "Say.Hello",
		"Content-Type": "application/protobuf",
	}
	Body: []byte(...)
}
```

