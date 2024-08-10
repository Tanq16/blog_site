---
title: Authentication and 2FA
date: 2020-10-13 12:00:00 +0500
categories: [Computers and Security]
tags: [security,authentication,mfa]
---

## Authentication

### Basic Authentication

It is the simplest authentication mechanism which is part of the HTTP protocol. It is a challenge response scheme where the server challenges the client to provide information to access a resource. The username and password is encoded using base64 and the value is set in the Authorization header and send it along in each HTTP request. Example &rarr; `Authorization: Basic XAAUVBBhHI87I==` . It is easy to implement and thus has faster APIs. In a non-SSL network, the encoding is basically useless. Sending the credentials in every HTTP request increases the attack surface.

### Digest based Authentication

This is an upgraded version of Basic Authentication. Instead of the base64 encoded strings, the server provides a digest for the client to use while encoding the username and password. Thus, the encoding is not universally known. When a client requests a resource without any authorization, the server sets the header with a certain digest information. The credentials are hashed using the digest and sent to the server along with the new request, which is decoded by the server and it grants the resource for valid credentials. This is still vulnerable to Man in The Middle attacks. It also requires 2 requests to a resource, the first of which is always wasted.

### Cookie/Session based Authentication

This is most commonly used in web apps. The credentials (username and password) are set in the parameters of a form and sent along in a request. The server validates the login and creates a cookie which is sent back in the response. The client uses this cookie to make future requests. This eliminates the need to send credentials in every request while enabling state maintenance in a stateless system. The validity of a cookie can be revoked anytime. It is generally useful for only Single domain systems. For multiple web apps and a separate client, the issue of CSRF arises. The session information is required to be stored in a database which questions the scalability.

### Token based Authentication

This is a form of stateless authentication. Instead of sending credentials for authentication, a server generated token is used. OAuth, JWT and OpenID are types of token based authentication.

**JWT &rarr;** When a user submits a username and password, the server validates it and returns a signed Jason Web Token. The token is used to allow future requests. The server doesn't bear the overhead of session information. It solves the issue of CSRF. However, the access to a user cannot be revoked and the safety of the token relies entirely on the consumer.

**OAuth2 &rarr;** This is an advanced token based authentication. Examples include signing with Facebook/Google/Apple. The user sends an authentication requests to Google/Facebook. Given the user has an account on Google or Facebook, the server responds with an authorization grant. Using this, the client requests an access token from the Authentication server. This token is sent to the application server, which validates the access token from the authentication server. Then the user is granted access to the resource.

**SSO &rarr;** Personal computers are a classic example of Single Sign On systems i.e., enter password once to get access to all the apps. Google is another example &rarr; A user trying to access Google forms sends a request to the forms service, which in turn calls the authentication service to make sure that the user is in fact logged in. If the user is not authenticated, then the service presents the login screen to authenticate the user. The advantage is that users need to remember only one password and it is secure because all services are being handled by one master authentication service.

## Two Factor Authentication

This is a subset of multi-factor authentication. TOTPs, Validation emails, Location (like GPay), Inherent factors such as Fingerprint and Iris scan, Knowledge based such as security questions are some of the factors used for 2FA/MFA. A threat model is essentially a structured representation of all the information that affects the security of an application. In essence, it is a view of the application and its environment through security glasses. It is a family of activities for improving security by identifying objectives and vulnerabilities, and then defining countermeasures to prevent, or mitigate the effects of, threats to the system.

### 2FA using TOTPs (Basic Implementation &rarr; Google Authenticator)

2FA is a new method for generating not normal OTPs, but **TOTPs (Time based One Time Passwords)**. Providers for 2FA services are &rarr; Google Authenticator, LastPass Authenticator, Microsoft Authenticator, Authy by Twilio, etc. TOTP is an algorithm that factors in the current time to generate a unique one-time password. TOTPs are secure because &rarr;

1. The password changes every n number of seconds (usually, 30 seconds), preventing eavesdroppers from using the same password later if they’re able to get a hold of it.
2. The password may be generated by an app on the user’s phone, making it more difficult for an attacker to acquire the password, as the user’s phone is usually by his/her side, unlike carrier messages which are susceptible to SIM theft.

TOTPs are user friendly because the user need only enter the password from an app or allow logins for integrated authenticators. TOTPs rely on the algorithm for **HMAC based OTPs (HOTPs)**. This document is an explanation on the implementation of both these algorithms as used by Google in Google Authenticator.

### Generating Secret

The first step is the creation of an application specific secret that will be used to verify the OTPs. This key will also be shared with the Authenticator app. It is using this secret key that the apps know what OTPs to generate for the authentication in apps or websites. One way of generating the secret is to generate a completely random buffer of data and encode it to base32. This can be done in NodeJS as follows &rarr;

```javascript
const crypto = require('crypto');
const base32 = require('hi-base32');

function generateSecret(length = 20) {
   const randomBuffer = crypto.randomBytes(length);
   return base32.encode(randomBuffer).replace(/=/g, '');
}
```

### Generating HOTPs

This step requires a secret key and a counter value. The secret is generated as shown in the above step. The counter value will be provided to the app when TOTPs are generated. The HOTP function accepts two arguments &rarr; `secret` and `counter`. The function first decodes the secret from base32. Then it creates a buffer from the counter value. A SHA-1 (in case of Google Authenticator) is used with the secret as the key and the buffer as the parameter. The HMAC result produced will be a 20 byte string. A 4 byte binary code is then extracted using **dynamic truncation**. Then, the first 6 digits of this code is extracted to get the final HOTP value. This process can be shown as a code block as follows &rarr;

```javascript
const crypto = require('crypto');
const base32 = require('hi-base32');

function generateHOTP(secret, counter) {
 const decodedSecret = base32.decode.asBytes(secret);
 const buffer = Buffer.alloc(8);
 for (let i = 0; i < 8; i++) {
    buffer[7 - i] = counter & 0xff;
    counter = counter >> 8;
 }

 // Step 1: Generate an HMAC-SHA-1 value
 const hmac = crypto.createHmac('sha1', Buffer.from(decodedSecret));
 hmac.update(buffer);
 const hmacResult = hmac.digest();

 // Step 2: Generate a 4-byte string (Dynamic Truncation)
 const code = dynamicTruncationFn(hmacResult);

 // Step 3: Compute an HOTP value
 return code % 10 ** 6;
}

function dynamicTruncationFn(hmacValue) {
 const offset = hmacValue[hmacValue.length - 1] & 0xf;

 return (
    ((hmacValue[offset] & 0x7f) << 24) |
    ((hmacValue[offset + 1] & 0xff) << 16) |
    ((hmacValue[offset + 2] & 0xff) << 8) |
    (hmacValue[offset + 3] & 0xff)
 );
}
```

### Generating TOTPs

The algorithm for generating TOTPs uses current time with a time-step of 30 seconds as the counter value in the `generateHOTP` function. The counter value can be generated as follows &rarr;

```javascript
const counter = Math.floor(Date.now() / 30000);
```

The function to generate the TOTP also accepts a window parameter which helps get the TOTP of any time window from the current time. Example &rarr; TOTP 2 mins back was at window=-4. The function can be written as follows &rarr;

```javascript
function generateTOTP(secret, window = 0) {
   const counter = Math.floor(Date.now() / 30000);
   return generateHOTP(secret, counter + window);
}
```

### Verification of TOTPs

The secret key is used to verify the TOTPs. The `generateTOTP` function is used to calculate the TOTP again and the value is checked to see if it matches or not. To improve experience of the user, the previous window's TOTP is also checked. Given variable `token` is the generated TOTP from the app, the verification function can be written as follows &rarr;

```javascript
function verifyTOTP(token, secret, window = 1) {
   if (Math.abs(+window) > 10) {
      console.error('Window size is too large');
      return false;
   }

   for (let errorWindow = -window; errorWindow <= +window; errorWindow++) {
      const totp = generateTOTP(secret, errorWindow);
      if (token === totp) {
         return true;
      }
   }

   return false;
}
```

## Resources

1. [Hacker Noon: How To Implement Google Authenticator Two Factor Auth in JavaScript](https://hackernoon.com/how-to-implement-google-authenticator-two-factor-auth-in-javascript-091wy3vh3)
2. [Security Models: Authentication and Authorization Explained](https://medium.com/@mikesparr/security-models-authentication-and-authorization-explained-7c227f228723)
