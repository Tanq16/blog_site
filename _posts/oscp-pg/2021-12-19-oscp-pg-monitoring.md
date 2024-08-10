---
title: OffSec PG - Monitoring
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.51.136`

### Network Scan

Nmap scan &rarr; `nmap -sC -sV -Pn -p- -A -o nmap.txt 192.168.51.136`

OS Detection &rarr; `Host: ubuntu; OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                    |
| -------- | ----------- | ------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0) |
| 25       | SMTP        | Postfix smtpd                                                 |
| 80       | HTTP        | Apache httpd 2.4.18 ((Ubuntu))                                |
| 389      | LDAP        | OpenLDAP 2.2.X - 2.3.X                                        |
| 443      | HTTPS       | Apache httpd 2.4.18 ((Ubuntu))                                |
| 5667     | Unknown     | \-                                                            |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.51.136 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed (for both http and https versions) &rarr;

- index.php
- javascript/
- nagios/

Visiting the web page reveals a `nagiosxi` directory which has a login.

---

## Exploitation

A web search for default admin credentials on nagios xi reveals `nagiosadmin:PASSW0RD`, however, after trial and error of easy passwords, `admin` was the correct password.

Searchsploit and web searches return a number of exploits for the nagios version. One of them is an authenticated RCE in the mointoring plugin upload capability.

Command used for searchsploit is as follows &rarr; `searchsploit nagios`, which returned many results. Spiraling down to RCE for version 5.6.5 (just above 5.6.0 and has root exploit), the full path can be received as follows &rarr; `searchsploit -p php/webapps/47299.php`. This gives the path and the exploit can be copied from there.

The exploit is basically a file upload where the name for the upload has an injection of commands such that it is executed in the backend. Therefore, a reverse shell can be executed on the backend, giving root access.

Setting up the exploit and calling it via the cli after setting up an `ncat` listener, a root shell is received. The flag of the root user can thus be read.

---

## Privilege Escalation

User was already root, therefore, no escalation was necessary.

---
