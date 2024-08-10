---
title: OffSec PG - FunBoxEasy
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.101.111`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.101.111`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.41 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.101.111 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- robots.txt
- index.html
- index.php
- profile.php (302)
- header.php
- registration.php
- logout.php
- dashboard.php (302)
- leftbar.php
- forgot-password.php
- hitcounter.txt
- icons/ (403)
- store/
- admin/
- secret/
- gym/

Robots txt file contains the disallwed entry for `gym`, which was already found by gobuster.

---

## Exploitation

Looking at the `store` directory, there is a book store with an admin login on the bottom of the page. This has the ability to add a new book which also has a file upload capability. This file is rendered as an image when visiting the book list via the publisher list. The publisher chosen for this was the Packt Publishing because it had 0 books.

The image being rendered can be navigated to separately at the location `/store/bootstrap/img/test.php`. The contents of the php file were first `<?php echo "Hello Test"; ?>` to test if the code is actually executed. This did execute upon visiting the aforementioned location, therefore, the code was replaced with the pentest monkey reverse shell.

This gives a reverse shell on the system when listening with netcat. This also gave the user flag.

---

## Privilege Escalation

### User

Looking at the `/etc/passwd` file, there are 2 users of interest &rarr; `tony` and `root`. Looking at the home directory of `tony`, it is readable by `www-data`. This directory had a `password.txt` file with the following contents &rarr;

```
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
```

The ssh password allows access to the machine as user `tony`.

### Root

Looked at the sudo capabilities of the user `tony`. The interesting entries were `whois`, `finger`, `time` and `cancel`. Out of these, only `time` was the tool which was already installed. `time` is a command that runs the command passed to it and records the time it took for the command to execute. Since sudo operation is allowed on it, it may escalate privileges while executing the shell.

Therefore, `sudo time /bin/bash` gives the root shell and thus the root flag.

---
