---
title: OffSec PG - Loly
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.225.121`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.225.121`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)** |
| -------- | ----------- | -------------------------- |
| 80       | HTTP        | nginx 1.10.3 (Ubuntu)      |
| 6311     | \-          | \-                         |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.225.121 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr; wordpress/

Busting the `/wordpress/` directory gives the following result &rarr;

- index.php (301)
- wp-content/
- wp-login.php
- license.txt
- wp-includes/ (403)
- readme.html
- wp-trackback.php
- wp-admin/ (302)

WPScan determines generic stuff and a user `loly`. Using this for a brute force attack with command `docker run -v /home/tanq/installations/SecLists/:/seclists/ -it --rm wpscanteam/wpscan --url http://192.168.225.121/wordpress/ -U loly -P /seclists/rockyou.txt` gives the credentials as `loly:fernando` for XMLRPC.

---

## Exploitation

Logging in to the `/wordpress/wp-login.php` as `loly` may work but does not move forward because of the requests being directed at `http://loly.lc` instead of the IP address. Therefore, this entry is added to `/etc/hosts` to enable navigation. Then the login page is visited and login is attempted as `loly`. This grants the admin page on the wordpress website.

The homepage has a plugin called `Adrotate` running. On this webpage, a section called "Manage Media". This says &rarr;

```
Accepted files: jpg, jpeg, gif, png, svg, html, js and zip. Maximum size is 512Kb per file.
Important: Make sure your file has no spaces or special characters in the name. Replace spaces
with a - or _. Zip files are automatically extracted in the location where they are uploaded
and the original zip file will be deleted once extracted. You can create top-level folders below.
Folder names can between 1 and 100 characters long. Any special characters are stripped out.
```

Therefore, uploading a reverse shell php code after zipping and uploading could allow navigating to it. The content is stored in the `/wordpress/wp-content/banners/` directory. Therefore, a shell is gained by listening on netcat and navigating to the php file in the said path. This is as the user `www-data` and this gives the user flag.

---

## Privilege Escalation

### User

Looking at the wordpress files, the `wp-config.php` file contains a password `lolyisabeautifulgirl`. Trying this password for `loly`, it works and grants the shell as `loly`.

### Root

Looking at the kernel version for the machine (4.4.0-31), there are a list of exploits. Starting with exploits from the highest 4.X version, one worked (45010.c). This gave the root shell and thus the root flag.

---
