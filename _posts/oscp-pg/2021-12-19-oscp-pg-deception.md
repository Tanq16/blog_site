---
title: OffSec PG - Deception
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.225.34`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.225.34`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.225.34 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- icons/ (403)
- wordpress/
- javascript/ (403)
- phpmyadmin/

Scanning the `/wordpress/` directory again gives the following &rarr;

- index.php (301)
- wp-content/
- wp-login.php
- license.txt
- wp-includes/
- readme.html
- robots.txt
- robots.html
- wp-trackback.php
- wp-admin/ (302)

Running WPScan on the target shows the following result &rarr;

- XMLRPC enabled at `/wordpress/xmlrpc.php`
- Directory listing at `/wordpress/wp-content/uploads/`
- WP-Cron enabled at `/wordpress/wp-cron.php`
- Wordpress version 5.3.2
- Enumerated users &rarr; `yash` and `haclabs`

---

## Exploitation

Looking at the page `/wordpress/robots.html`, it has a click interface which shows a playful alert. Looking at the code, a new webpage `/wordpress/admindelete.html` was discovered. This page says `LOL,A Noob is looking for a hint`. Based on this, searching for `/wordpress/hint.html` was discovered, which says `Please collect all the API tokens availabe on the home page`. Therefore, the homepage was scoured for API tokens. The following were found &rarr;

```
API old0 : 5F4DCC3B5AA
API old1 : 765D61D8
API old2 : 327DEB
API new : 882CF99
```

When concatenated, this gives `5F4DCC3B5AA765D61D8327DEB882CF99`. This is not any kind of hash, therefore, trying this out in credentials `yash:5F4DCC3B5AA765D61D8327DEB882CF99` works. This gives the shell as user `yash` and the local flag.

---

## Privilege Escalation

### User

The uuser `yash` is not allowed to run `sudo`, therefore, looked at the setuid binaries. The interesting ones were `/usr/bin/arping` and `/usr/bin/traceroute6.iputils`. Looking at the files in the home directory, there is a file `.systemlogs` which contains a bunch of text. This does have the username `haclabs` within `""`. Grepping out `"` to accentuate them shows the following values &rarr;

```
haclabs
A=123456789
+A[::-1]
```

The following values make sense from the above &rarr; `987654321`, `123456789987654321` and `haclabs987654321`. Trying these out for the user `haclabs`, the last one works and grants the shell.

### Root

`haclabs` can execute `sudo` for all commands without a password. Using this to spawn a shell grants the shell as `root` and thus, the root flag.

---
