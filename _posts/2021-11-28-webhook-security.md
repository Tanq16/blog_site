---
title: Basics of WebHook Security
date: 2021-11-27 12:00:00 +0500
categories: [Web Application Security]
tags: [webhook,web-application,security,http]
---

# Introduction

Apps can communicate with each other in one of two ways &rarr; Polling or through WebHooks. WebHooks are automated messages sent by an app when something happens. They have a payload and are sent to a unique URL (or phone number) that belongs to the app being communicated with. This is faster than polling and requires less effort to develop.

The consumer of the information provided via a WebHook must tell the originating app, the address of the WebHook, where the data will be received. The simplest way to do so is through an HTTP request, preferably a POST request.

WebHooks typically connect two applications. They are similar to APIs but are much simpler.

# Security Issues in WebHooks

1. Man in the Middle against HTTP data sent by WebHooks &rarr; Must always use TLS for exchange of any public data
2. Forgery of WebHook requests &rarr; Verification in form of a hash passed in a header such as `X-<vendor>-hmac-SHA256` must be added to prevent this type of attack
3. Lack of authentication &rarr; Can be fixed by adding a basic authentication header with a client generated secret over TLS
4. Replay attacks &rarr; This can be prevented by ensuring idempotency of endpoints or by adding timestamped signatures for the `hmac`

Security Solutions captured in tabular form is as follows &rarr;

## Table

| Threat                               | Solution                                               |
| ------------------------------------ | ------------------------------------------------------ |
| Payload Exposure                     | HTTPS WebHook URLs with SSL Encryption                 |
| Attack from unknown WebHook sources  | Authentication token, IP whitelists for WebHook source |
| WebHook interception and redirection | Client verification using Mutual TLS                   |
| WebHook payload corruption           | Message verification using HMAC signatures             |
| Replay attacks                       | Timestamped messages                                   |

---

# Resource

[Hookdeck](https://hookdeck.com)
