---
title: OffSec PG - FunBoxEnum
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.244.132`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.244.132`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.244.132 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- icons/ (403)
- javascript/ (403)
- mini.php
- robots.txt
- phpmyadmin/

Robots txt file lists `Enum_this_Box` as allowed.

---

## Exploitation

The webpage `mini.php` is actually a shell interface with some limited options. This was used to read the local flag. This also had the functionality to upload files. This was used to upload the pentest monkey's reverse shell code and get full shell.

---

## Privilege Escalation

### User

Looked at the `/etc/passwd` file, the users of importance are &rarr; `root`, `goat`, `harry`, `karla`, `lissy`, `sally` and `oracle`. The user `oracle`'s entry is &rarr; `oracle:$1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0:1004:1004:,,,:/home/oracle:/bin/bash`. Using John the Ripper to crack this gives the credentials `oracle:hiphop`. This did not work for ssh though.

Running `hydra` to brute-force other users by using &rarr; `hydra -l karla -P /home/tanq/installations/SecLists/rockyou.txt ssh://192.168.225.132`. Ran linenum script for more enumeration, however, nothing useful was discovered. Tried user:user form of credentials for all users, which gave a success for `goat:goat`. This gives the user shell as user `goat`.

### Root

Looking at `sudo -l` to enumerate privilege of the `goat` user, they are allowed to run `/usr/bin/mysql` as root without any password. MySQL has a functionality for spawning a shell. Executing `sudo /usr/bin/mysql` gives the mysql shell and using `\! bash` in the mysql shell spawns a bash shell as the user `root`. This gives the root flag.

---
