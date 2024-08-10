---
title: OffSec PG - Cyberspoloit1
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.124.92`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.124.92`

OS Detection &rarr;  `os_info`

| **Port** | **Service** | **Other details (if any)**                                    |
| -------- | ----------- | ------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.2.22 ((Ubuntu))                                |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.124.92 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed &rarr;

- index/
- index.html
- robots/
- hacker/

The `hacker/` page consists of a base64 string which decodes to `cybersploit{youtube.com/c/cybersploit}`.

---

## Exploitation

The source of the home page consists of `<!-------------username:itsskv--------------------->`. Therefore, this can be used for a password brute force on the ssh server. This did not reveal a password. Testing the previous base64 string as the password works and grants access to the user `itsskv`.

This gives the first flag.

---

## Privilege Escalation

Enumerating the setuid binaries gives no info. Looking at the kernel version and os version using `uname -a`, it seems the kernel is indeed an outdated one &rarr; 3.13.0. Using searchsploit to look at exploits for this version using `searchsploit 3.13.0`, there is an overlay.fs exploit for local privilege escalation.

Compiling this binary for the 32 bit version and transferring to the machine via wget, enables getting `root` user shell after running it. This gives the root flag.

---
