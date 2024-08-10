---
title: OffSec PG - Y0usef
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.244.138`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.244.138`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                      |
| -------- | ----------- | --------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.10 ((Ubuntu))                                  |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.244.138 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.php
- icons/ (403)
- administration/ (403)

---

## Exploitation

Without much information from the web scan, the possibility to look at headers was apparent. Adding the `X-Forwarded-For: 192.168.244.138` header allows loading of the internal `/administration/` directory. The header needs to be added to all subsequent requests via burp intercept.

This directory has a login page which does not have SQLi type injections, however, default credentials of `admin:admin` work. The dashboard of the application gives the ability to upload a file, list users or log out. The `.../users` page does not list any useful information.

Note: There was a spelling error in the links, which needed to be modified to get correct response.

The interesting part was the upload functionality. It could be used to upload a reverse shell. However, the application does not directly allow php files. Bypassing this was checked by renaming the file and adding image headers to the content, but it didn't work.

The thing that worked was modifying the `Content-Type` header to `image/gif`. This allowed the upload of the reverse shell along with the path of the uploaded file. Navigating to the file grants a shell as the `www-data` user over netcat. This also gives the user flag.

---

## Privilege Escalation

### User

Enumerating the `/etc/passwd` file, the users of importance are `root`, `yousef` and `speech-dispatcher`. The `/home/` directory contains a file `user.txt` file which has a base64 encoded string `c3NoIDogCnVzZXIgOiB5b3VzZWYgCnBhc3MgOiB5b3VzZWYxMjM=`.

Decoding this gives the credentials `yousef:yousef123`. Using this with ssh gives the shell as user `yousef`.

### Root

Checking the `sudo -l` capabilities of `yousef`, it shows that `yousef` may run any command as root using `sudo`. Therefore, `sudo su` grants the root shell and thus, the root flag.

---
