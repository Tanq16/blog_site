---
title: Fundamentals of SSL/TLS
date: 2020-01-18 12:00:00 +0500
categories: [Web Application Security]
tags: [http,transport,web-application,security,ssl,tls]
---

# Introduction

**Secure Socket Layer (SSL)** or **Transport Layer Security (TLS)** is a protocol for secure transmission of data. It offers **confidentiality**, **integrity** and **authentication** by encrypting and signing messages along with a **handshake** process for authentication. It also detects missing and duplicate messages. It is the primary way to secure web traffic. SSL is the older protocol and is broken (all 3 versions). SSLv3 is broken by POODLE. TLS is the newer standard.

## Description

Client Server applications use TLS to protect their communications. They can be in plain-text as well, so to differentiate, the service associates it with different port numbers (for example &rarr; HTTP on 80, HTTP over TLS or HTTPS on 443). Services like news protocol, mail servers and web servers have a mechanism to allow clients to switch to the TLS version by using **STARTTLS** function. Once the client and server have agreed to use TLS, they negotiate a stateful connection using a handshake procedure with an asymmetric cipher to establish cipher settings and a session specific shared symmetric key for further communication.

## Digital Certificates

A digital certificate certifies the ownership of a public key. This allows others to rely upon signatures or on assertions made by the private key that corresponds to the certified public key. TLS typically relies on a set of trusted third-party certificate authorities to establish the authenticity of certificates. Trust is usually anchored in a list of certificates distributed with user agent software, and can be modified by the relying party. As a consequence of choosing X.509 certificates, certificate authorities and a public key infrastructure are necessary to verify the relation between a certificate and its owner, as well as to generate, sign, and administer the validity of certificates.

## Algorithms

Before a client and server can begin to exchange information protected by TLS, they must securely exchange or agree upon an encryption key and a **cipher** to use when encrypting data. For the encryption key, the public and private keys are used in a **key exchange algorithm**. Several available algorithms for key exchange are &rarr; RSA, DH-RSA, DHE-RSA (forward secrecy), ECDH-RSA, ECDHE-RSA (forward secrecy), DH-DSA, ECDH-DSA, PSK, PSK-RSA, etc. TLS v1.3 only allows the forward secrecy variants. Several ciphers available are &rarr; *AES-GCM*, *AES-CCM*, AES-CBC, Camellia-CBC, Camellia-GCM, ARIA-GCM, RC4, *ChaCha20-Poly1305*, etc. TLS v1.3 only has the italicized ciphers. Several hashing (MAC) algorithms available are &rarr; HMAC-MD5, HMAC-SHA1, HMAC-SHA256/384, AEAD and a few more. TLS v1.3 only has AEAD (which doesn't use an HMAC).

## Libraries

Some of the libraries for SSL are &rarr; BoringSSL, Delphi, GnuTLS, cryptlib, OpenSSL, mbedTLS.

## Other uses

SMTP (Simple Mail Transfer Protocol) can be protected by TLS (use of public key certificates to verify the identity of the end points). It can also be used to tunnel an entire network stack to create a VPN (OpenVPN and OpenConnect). Compared to traditional IPsec VPN technologies, TLS has some inherent advantages in firewall and NAT traversal that make it easier to administer for large remote-access populations.

# Security

The most secure version of SSL is SSL v3 which improved upon its predecessors by adding SHA-1 based ciphers. It was broken in 2014 by the **POODLE** attack, where the CBC mode of operation became vulnerable to a padding attack. **Attacks against SSL/TLS**

1. Protocol Downgrade Attacks &rarr; A version rollback attack tricks a web server into negotiating connections using previous versions of TLS (like SSL v2) that have been abandoned as insecure. **FREAK** is a downgrade attack on the encryption keys. The attack involved tricking servers into negotiating a TLS connection using cryptographically weak 512 bit encryption keys. **Logjam** exploits the option of using legacy 512-bit Diffie-Hellman groups dating back to the 1990s.
2. The **DROWN** attack is an exploit that attacks servers supporting contemporary SSL/TLS protocols by exploiting their support for the obsolete and insecure SSLv2 to attack connections using latest protocols that would otherwise be secure.
3. **BEAST (Browser Exploit Against SSL/TLS)** attack on TLS v1 was a PoC for a long known vulnerability of CBC &rarr; An attacker observing 2 consecutive ciphertext blocks C0, C1 can test if the plaintext block P1 is equal to x by choosing the next plaintext block P2 = x ^ C0 ^ C1; as per CBC operation, C2 = E(C1 ^ P2) = E(C1 ^ x ^ C0 ^ C1) = E(C0 ^ x), which will be equal to C1 if x = P1.
4. **POODLE (Padding Oracle on Downgraded Legacy Encryption)** &rarr; On average, attackers only need to make 256 SSL 3.0 requests to reveal one byte of encrypted messages.
5. Implementation Attacks &rarr;
	- **Heartbleed** bug &rarr; It was a serious bug in the implementation of OpenSSL that allowed attackers to steal private keys from from servers using a buffer overread vulnerability.
	- **BERserk** bug &rarr; Daniel Bleichenbacher's PKCS#1 v1.5 RSA signature forgery is a result of incomplete ASN.1 length decoding of public key signatures which allows MiTM by forging the public key signature.
	- **Cloudbleed** bug &rarr; This is an implementation error caused by a single mistyped character in code used to parse HTML which created a buffer overflow error on Cloudflare servers. This allowed unauthorized third parties to read data in the memory of programs running on the servers, data that should otherwise have been protected by TLS.

# TLS Handshake

The TLS protocol exchanges **records**, which encapsulate the data to be exchanged in a specific format. Each record can be compressed, padded, appended with a MAC, or encrypted, all depending on the state of the connection. Each record is a TLV vector where value is the version of TLS being used. The data encapsulated may be control or procedural messages of the TLS itself, or the application data needed to be transferred by TLS. When the connection starts, the record encapsulates a "control" protocol, the handshake messaging protocol (content type 22). This protocol is used to exchange all the information required by both sides for the exchange of the actual application data by TLS. This initial exchange results in a successful TLS connection or an alert message.

## TLS Handshake (RSA Key Exchange)

In this handshake, the server is authenticated by its certificate but the client is not. The procedure is as follows &rarr;

1. The client sends the **ClientHello** message specifying the highest TLS version it supports, a random number, a list of suggested cipher suites and a list of suggested compression methods. If this is meant to be a *resumed handshake*, the client may send a session ID. If the client can use ALPN, it may include a list of supported protocols like HTTP/2.
2. The server responds with a **ServerHello** message, containing the chosen version, a random number, chosen cipher suite and the chosen compression method. To confirm a resumed handshake, the server may send a session ID. The chosen version is intersection_max(client-supported, server-supported).
3. The server sends its certificate (may be omitted depending on the cipher suite), which is verified by the client with the CA that issued it. The server then sends a **ServerHelloDone** message.
4. The client responds with a **ClientKeyExchange** message, which may contain a **PreMasterSecret** (encrypted using the server's public key), public key, or nothing, depending on the cipher suite.
5. The client and server then use the random numbers and the **PreMasterSecret** to compute a common secret called the **master secret**. All other data like session keys, IV, symmetric encryption key, MAC key for the connection are derived using this secret along with the random numbers.
6. The client now sends a **ChangeCipherSpec** record, telling the server, "Everything I tell you from now on will be authenticated (and encrypted if encryption parameters were present in the server certificate)." The ChangeCipherSpec is a record-level protocol with content type 20. The client sends an authenticated and encrypted **Finished** message, containing a hash and MAC over the previous handshake messages. The server attempts to decrypt the Finished message and verify the hash and MAC. If the decryption or verification fails, the handshake also fails and the connection is torn down.
7. Finally, the server sends a ChangeCipherSpec, telling the client, "Everything I tell you from now on will be authenticated (and encrypted, if encryption was negotiated)." The server sends its authenticated and encrypted Finished message. The client performs the same decryption and verification procedure.
8. Application messages exchanged between client and server from this point will also be authenticated and optionally encrypted exactly like in the Finished message.

## Client Authenticated TLS Handshake (RSA Key Exchange)

1. The client sends the **ClientHello** message specifying the highest TLS version it supports, a random number, a list of suggested cipher suites and a list of suggested compression methods. If this is meant to be a *resumed handshake*, the client may send a session ID. If the client can use ALPN, it may include a list of supported protocols like HTTP/2.
2. The server responds with a **ServerHello** message, containing the chosen version, a random number, chosen cipher suite and the chosen compression method. To confirm a resumed handshake, the server may send a session ID. The chosen version is union_max(client-supported, server-supported).
3. The server sends its certificate (may be omitted depending on the cipher suite), which is verified by the client with the CA that issued it.
4. The server sends a **CertificateRequest** message to get the client's certificate. Then, it sends a **ServerHelloDone** message.
5. The client responds with a **Certificate** message which includes it's certificate. It then sends a **ClientKeyExchange** message, which may contain a **PreMasterSecret** (encrypted using the server's public key), public key, or nothing, depending on the cipher suite.
6. The client then sends a **CertificateVerify** message, which is a signature over the previous handshake messages using the client's private key, which is verified by the server using the client's public key.
7. The client and server then use the random numbers and the **PreMasterSecret** to compute a common secret called the **master secret**. All other data like session keys, IV, symmetric encryption key, MAC key for the connection are derived using this secret along with the random numbers.
8. The client now sends a **ChangeCipherSpec** record, telling the server, "Everything I tell you from now on will be authenticated (and encrypted if encryption parameters were present in the server certificate)." The ChangeCipherSpec is a record-level protocol with content type 20. The client sends an authenticated and encrypted **Finished** message, containing a hash and MAC over the previous handshake messages. The server attempts to decrypt the Finished message and verify the hash and MAC. If the decryption or verification fails, the handshake also fails and the connection is torn down.
9. Finally, the server sends a ChangeCipherSpec, telling the client, "Everything I tell you from now on will be authenticated (and encrypted, if encryption was negotiated)." The server sends its authenticated and encrypted Finished message. The client performs the same decryption and verification procedure.
10. Application messages exchanged between client and server from this point will also be authenticated and optionally encrypted exactly like in the Finished message.

## TLS Handshake (Ephemeral Diffie Hellman Key Exchange)

1. The ClientHello like in the RSA variant.
2. The ServerHello message, like in RSA, contains the server certificate, cipher suite and the server random. The message also includes the server's DH parameter and the client random. The client random, DH parameter and the server random are all encrypted with the server private key and sent as a digital signature. This is the part of the ServerKeyExchange message for the DH-based protocol. This is verified by the client.
3. After verification of the signature, the client sends it's DH parameter to the server. Instead of the client generating the premaster secret like in the RSA variant, the client and server use the DH parameters to obtain the matching premaster secret.
4. Everything else is the same.

## Keyless TLS Handshake

Keyless SSL is a service for companies that use a cloud vendor for SSL encryption. In such a case, the cloud vendor would need to know the company's private key. Keyless SSL is a way to circumvent that. By moving the part of the handshake involving the private key off of the vendor's server, the private key can remain securely in the company's possession. Instead of using the private key directly to generate session keys, the cloud vendor gets the session keys from the company over a secure channel and uses those keys to maintain encryption. Keyless TLS makes use of the fact that there is only one time when the private key is used during the handshake. When necessary, the vendor's server forwards the data to the company's private server which uses the key as required and sends the result back, after which usual TLS handshake continues. The step that requires the private key is when the server decrypts the Premaster Secret in the RSA variant. In the EDH variant, it is used to construct the digital signature for the random value and the server DH parameter.

## TLS v1.3 Handshake

The TLS 1.3 handshake was condensed to only one round trip compared to the two round trips required in previous versions of TLS/SSL. The client sends the ClientHello and makes a guess on what key algorithm will be used so that it can send a secret key to share if needed. This way, it reduces the round trip made by the server.

## Resumed TLS Handshake

Public key operations are relatively expensive in terms of computational power. TLS provides a secure shortcut in the handshake mechanism to avoid these operations &rarr; **resumed sessions**, which are implemented using session IDs or session tickets. Apart from the performance benefit, they also enable Single Sign On (SSO) because it can guarantee that both the original and resumed session originate from the same client. **Session IDs and Session Tickets &rarr;** In an ordinary full handshake, the server sends a session id as part of the ServerHello message. The client associates this session id with the server's IP address and TCP port, so that when the client connects again to that server, it can use the session id to shortcut the handshake. In the server, the session id maps to the cryptographic parameters previously negotiated, specifically the master secret. The use of session tickets, instead of session IDs defines a way to resume a TLS session without requiring that session-specific state is stored at the TLS server. The TLS server stores its session-specific state in a session ticket and sends the session ticket to the TLS client for storing. The client resumes a TLS session by sending the session ticket to the server, and the server resumes the TLS session according to the session-specific state in the ticket. The session ticket is encrypted and authenticated by the server, and the server verifies its validity before using its contents.

# SSL Certificates

SSL certificates include &rarr;

- The domain name that the certificate was issued for
- Which person, organization, or device it was issued to
- Which certificate authority issued it
- The certificate authority's digital signature
- Associated subdomains
- Issue date of the certificate
- Expiration date of the certificate
- The public key (the private key is kept secret)

For an SSL certificate to be valid, domains need to obtain it from a certificate authority (CA). A CA is an outside organization, a trusted third party, that generates and gives out SSL certificates. The CA will also digitally sign the certificate with their own private key, allowing client devices to verify it. Most, but not all, CAs will charge a fee for issuing an SSL certificate. Technically, anyone can create their own SSL certificate by generating a public-private key pairing and including all the information mentioned above. Such certificates are called self-signed certificates because the digital signature used, instead of being from a CA, would be the website's own private key.

## CRLs (Certificate Revocation Lists)

CRL is a list of digital certificates which have been revoked by the issuing CA before their scheduled expiration date and shouldn't be trusted. There are 2 revocation states &rarr; **Revoked** (irreversible revocation if the certificate is improperly issued or the private key is stolen) and **Hold** (for temporary invalidation in cases when the user is unsure if the key has been lost). A CRL is generated and published periodically, often at a defined interval. It can also be published immediately after revocation of a certificate. A CRL is generally issued by a CRL issuer which is typically the issuer of the certificates but can also be another trusted third party. CRLs have a lifetime of ≤ 24 hours. To prevent spoofing or DoS attacks, CRLs carry a digital signature associated with the CA that publishes it. The problem with CRL is that every time the certificate is to be checked for validity, there should be access to the CRLs. The alternative is to use a certificate validation protocol, OCSP. The CRL is downloaded every time by the client after it connects to the CA.

# OCSP (Online Certificate Status Protocol)

This is a protocol to obtain the revocation status of a certificate. The request/response is generally communicated over HTTP. It contains less data than a CRL which puts less burden on the network and the client resources. Due to less data to be parsed in a response, the libraries that handle it can be less complex. The PKI implementation in basic terms is as follows &rarr;

1. Alice and Bob have certificates issued by Carol, a CA. Alice wants to perform a transaction with Bob and thus sends him her public key. Bob is worried that the private key of Alice may have been compromised.
2. Bob creates an OCSP request which contains the serial number of Alice's certificate and sends it to Carol (who issued the certificates).
3. Carol's OCSP responder reads the serial number and verifies the status of the certificate by looking at the CA database (CRL issuer database). The responder then returns a signed OCSP response to Bob, who cryptographically verifies Carol's signature. Then the transaction can take place. Browser providers started maintaining own lists for revoked certificates.

## Details of the protocol

OSCP can have replay attacks because the response that marks a certificate as good can be recorded and sent again at a later time to validate the certificate after it has been revoked. Thus, a nonce can be included in the request which must be present in the response. But, this is not used to prevent load on the OCSP responders, instead pre-signed responses with a validity of multiple days are used. So, replay attacks are a major vulnerability. The key that signs a response need not be the same key that signed the certificate. The certificate's issuer may delegate another authority to be the OCSP responder.

## OCSP stapling

This uses OCSP in an advanced way where it places the responsibility to perform the checks on the validation of the certificates on the web server instead of the client. It works in the following way &rarr;

1. The web server sends regular, automatic OCSP requests to the OCSP responder (CA), which can be as frequent as few hours.
2. The responder provides a timestamped response and a CA signed approval message. This response is cached by the web server, which it uses until the next update from the responder.
3. The web server sends the cached response and timestamped data to the client, when the client attempts to connect to the server. This response is essentially *stapled* when the server sends the TLS certificate during the handshake.

Clients don't always know if the server will staple the response from the responder or not. Therefore, OCSP Must-Staple is an extension which alerts the client if it should expect a stapled response.
> When an intermediate CA certificate is revoked, the client (or server in the case of stapling) must check the CRLs (or query OCSP) in succession along the chain of trust up to the root CA.

# HSTS (HTTP Strict Transport Security)

In 2009 at BlackHat DC, Moxie Marlinspike presented a tool known as **SSLStrip**. This tool would intercept HTTP traffic and whenever it spotted redirects or links to sites using HTTPS, it would transparently strip them away. Instead of the victim connecting directly to a website; the victim would connect to the attacker, and the attacker would initiate the connection back to the website. This attack is known as an **on-path attack**. The magic of SSLStrip was that whenever it would spot a link to a HTTPS webpage on an unencrypted HTTP connection, it would replace the HTTPS with a HTTP and sit in the middle to intercept the connection. The interceptor would make the encrypted connection back to the web server in HTTPS, and serve the traffic back to the site visitor unencrypted. In response, a protocol called HTTP Strict Transport Security (HSTS) was created in 2012. The protocol works by the server responding with a special header called `Strict-Transport-Security` which contains a response telling the client that whenever they reconnect to the site, they must use HTTPS. The response also contains a `max-age` field which defines how long the rule should last for since it was last seen. HSTS Preload Lists are one potential solution to help with these issues, they effectively work by hardcoding a list of websites that need to be connected to using HTTPS-only. They can be submitted to the Chrome HSTS Preload List at `hstspreload.org`.

## HSTS Preloading

One of the shortcomings of HSTS is the fact that it requires a previous connection to know to always connect securely to a particular site. When the visitor first connects to the website, they won't have received the HSTS rule that tells them to always use HTTPS. Only on subsequent connections will the visitor's browser be aware of the HSTS rule that requires them to connect over HTTPS. Other mechanisms of attacking HSTS have been explored, for example &rarr; by hijacking the protocol used to the sync a computer's time (NTP), it can be possible to set a computers date and time to one in the future. This date and time can be set to a value when the HSTS rule has expired and thereby bypassing HSTS. Leonardo Nve revived SSLStrip in a new version called SSLStrip+, with the ability to avoid HSTS. When a site is connected to over an unencrypted HTTP connection, SSLStrip+ will look for links to HTTPS sites. When a link is found to a HTTPS site, it is rewritten to HTTP and critically the domain is rewritten to an artificial domain which is not on the HSTS Preload list. For example &rarr; suppose a site contains a link to `https://example.com/`, the HSTS could be stripped by rewriting the URLs to `http://example.org/`, with attacker sitting in the middle, receiving traffic from `http://example.org/` and proxying it to `https://example.com/`.

# Resources

1. [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security)
2. [Online Certificate Status Protocol](https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol)
3. [Performing & Preventing SSL Stripping: A Plain-English Primer](https://blog.cloudflare.com/performing-preventing-ssl-stripping-a-plain-english-primer/)
4. [Everything You Need to Know About OCSP, OCSP Stapling & OCSP Must-Staple](https://www.thesslstore.com/blog/ocsp-ocsp-stapling-ocsp-must-staple/)
