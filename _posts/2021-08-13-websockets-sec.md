---
title: Fundamentals of Websockets and their Security
date: 2021-08-13 12:00:00 +0500
categories: [Web Application Security]
tags: [http,websocket,transport,web-application,security]
---

# Test Methodology

## Capture, Discover or Analyze

1. Use Burp to capture packets and search for connection upgrades. The proxy tab has a WebSockets History sub-tab for looking at the communication.
2. 101 response code can be used to search for WebSocket upgrades in Burp.
3. A message can be sent to the repeater for replaying and generating new messages.
4. The handshake process can also be manipulated by using the "pencil" icon in repeater next to the socket URL. This allows attaching to an existing connected WebSocket, clone a connected WebSocket or reconnect to a disconnected WebSocket.
5. If a new connection is established, repeater can be used to send new messages.

## Vulnerabilities

1. User supplied input transmitted via WebSockets could lead to SQLi, XSS, XXE or any other client side vulnerabilities. Therefore, check for sanitization. Similarly validate server data before publishing into DOM.
2. WebSockets are not affected by SOP, therefore, implementations that rely on session cookies to perform authenticated actions could lead to CSWH (Cross Site WebSocket Hijacking).
3. WebSockets do not have any particular way for servers to authenticate clients during handshake and therefore, rely on other solutions being implemented that make use of cookies, JWTs or HTTP Auth. Similarly, there is a lack of authorization.
4. WebSockets in general use the `Origin` header for differentiating between connections. HTTP servers also use it to prevent CSRF because CORS is not supported by WebSockets. Client side javascript is not allowed by modern browsers to modify the Origin header, therefore, it is infeasible for an attacker to modify the header on the victim’s browser to bypass this check.
5. Not using TLS i.e., `wss` sockets instead of `ws` sockets can be harmful for integrity and confidentiality.
6. WebSockets allow an unlimited number of connections to reach the server, which could allow for DoS attacks. Therefore, rate limiting is required. Leaky bucket is a good also to use.
7. Avoid tunneling arbitrary TCP connections through WebSockets. Example &rarr; database connection through the browser. This could allow escalation for an in-browser (XSS executed) attacker.
8. Resource hogging is possible if there is no restriction on payload size for WebSocket messages. This must be implemented on the server side library for WebSockets.
9. Use a robust communication protocol. This prevents or at least mitigates WebSocket hijacking by logging info such as IP address in case the format of messages used by the sender differs from that specified by the implemented "robust" protocol.

---

# Example Scenarios

## Ticket based authentication system

This is a common implementation to solve WebSocket authentication problem. The working is as follows &rarr;

1. Client code contacts the HTTP server to obtain an authorization ticket before opening a WebSocket.
2. The server generates the ticket, which contains some form of an ID, IP of the client that requested it, a timestamp, and some other record keeping parameters.
3. The server stores this ticket in a database or a cache and also returns it to the client.
4. The client opens a WebSocket and sends the ticket as part of the initial handshake in HTTP (possibly via a header).
5. The server verifies the ticket for reuse, IP, expiration or an auth check. If it validates, then it allows the WebSocket to be established.

[SignalR](https://docs.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr) also provides WebSockets, to be implemented on Web App Owners' server or as a passthrough via SignalR’s domain, with possibility of the above authentication system.

---

# Concepts

WebSockets are bi-directional, full duplex communications protocol initiated over HTTP, used for streaming data and other asynchronous traffic. Typically, the server simultaneously handles WebSocket and traditional HTTP requests from the browser client. But, WebSockets can also be used in any other client-server application, for e.g. with desktop applications.

**How is it different from HTTP?** &rarr; Unlike the transaction type flow of HTTP (request followed by response, then a new transaction takes place), WebSockets are established over HTTP and are typically long-lived, with the messages not being transactional i.e., can be sent in either direction at any time. A connection typically stays open and idle till one side is ready to send some data.

## WebSocket Handshake

A connection can be created using client-side javascript like &rarr;

```
var ws = new WebSocket("wss://website.com/application");
```

A `ws` protocol uses an unencrypted connection, while the `wss` protocol establishes a socket over an encrypted TLS connection. To establish a WebSocket connection, the client and server perform a handshake over HTTP.

**Request by client** &rarr;

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: keep-alive, Upgrade
Sec-WebSocket-Key: d2Vic29ja2V0c2FyZWdvb2QhCg==
Sec-WebSocket-Protocol: chat
Sec-WebSocket-Version: 13
Cookie: session=c29ja2V02i2jaVcFyZW02FyZvbQhCgc2FyZWd
Origin: http://example.com

```

**Response by server** &rarr;

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
Sec-WebSocket-Protocol: chat
```

**Headers used** &rarr;

1. Sec-WebSocket-Protocol &rarr; This signifies version and is usually 13.
2. Upgrade and Connection &rarr; Indicate WebSocket establishment.
3. Sec-WebSocket-Key &rarr; Request header that contains a base64 encoded random value, generated in each handshake request.
4. Sec-WebSocket-Accept &rarr; Response header that contains the base64 encoded result of the SHA-1 hash of the value in Sec-WebSocket-Key header, concatenated with the fixed string `258EAFA5-E914-47DA-95CA-C5AB0DC85B11` (a GUID) defined in the protocol. This is intended to prevent a caching proxy from replaying a WebSocket conversation. This does not provide authentication, integrity or privacy.

The handshake resembles HTTP to allow servers to handle HTTP and WebSocket connections on the same port. After a connection is established, communication switches to a bidirectional binary protocol which doesn't conform to the HTTP protocol.

## Communication

Once a connection is established, messages can be sent asynchronously in either direction. The client can send a message using javascript as follows &rarr;

```
ws.send("Test message");
```

Theoretically, WebSocket messages can contain any content or format, but in modern applications, it is common to use JSON structured data within messages. For e.g. &rarr;

```
{"user":"A person", "content":"This is a test message"}
```

The data is minimally framed, with a small header followed by payload. WebSocket transmissions are described as "messages”, where a single message can optionally be split across several data frames. This allows sending messages where initial data is available but the complete length of the message is unknown (it sends one data frame after another until the end is reached and marked with the FIN bit). With extensions to the protocol, this can also be used for multiplexing several streams simultaneously (for instance to avoid monopolizing use of a socket for a single large payload).

---

# Resources

1. [PortSwigger: WebSockets](https://portswigger.net/web-security/websockets)
2. [Heroku: WebSocket Security](https://devcenter.heroku.com/articles/websocket-security)
3. [AppKnox: WebSocket Pentesting](https://www.appknox.com/blog/everything-you-need-to-know-about-web-socket-pentesting)
4. [FreeCodeCamp: Securing WebSockets](https://www.freecodecamp.org/news/how-to-secure-your-websocket-connections-d0be0996c556/)
5. [VaaData: WebSockets Security Risks](https://www.vaadata.com/blog/websockets-security-attacks-risks/)
6. [OWASP: Testing WebSockets](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/11-Client_Side_Testing/10-Testing_WebSockets)
