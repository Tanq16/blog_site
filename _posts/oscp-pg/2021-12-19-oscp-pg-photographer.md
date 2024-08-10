---
title: OffSec PG - Photographer
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.59.76`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.59.76`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                    |
| -------- | ----------- | ------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.18 ((Ubuntu))                                |
| 139      | NETBIOS-SSN | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)                   |
| 445      | NETBIOS-SSN | Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)               |
| 8000     | HTTP-ALT    | Apache/2.4.18 (Ubuntu)                                        |

### Web Scan

GoBuster scan &rarr; `/opt/GoBuster/gobuster dir -u http://192.168.59.76 -w /opt/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- generic.html
- elements.html
- images/
- assets/

Scanning the web site on port 8000 with a `-x html,php,txt` and a `-f` flag gives the following &rarr; 

- app/
- admin/
- index/
- set/

Running nmap to discover shares using `nmap --script smb-enum-shares -p 139,445 192.168.59.76` provided with the shares of `IPC`, `sambashare`, and `print`. This also showed the capability to read and write using anonymous users and a possible username of `agi`.

---

## Exploitation

Listing Samba shares using `sambaclient -U '' //192.168.59.76/sambashare`. This share had an email to user `daisa` and a backup of a wordpress site. The email contained a hint to a possible password for `daisa`, using that on the website at the `/admin` path for the website at port 8000 provided access with credentials `daisa:babygirl`.

Here, the Library segment contains a file upload functionality that is interesting. Image files can be uploaded and can be previewed in the application. Messing with this functionality to upload a PHP code by changing the `Content-Type` to `application/php` and the filename to include the `.php` extension with the data containing the following code &rarr; 

```php
<?php
echo("hacked");
?>
```

This file can be uploaded without any issue which means the server is vulnerable to arbitrary file upload. The preview for this php file doesn’t work of course, but the application shows a “Download File” option due to the failed preview. This button contains a link to the file at `http://192.168.59.76:8000/storage/originals/50/ce/cyberpunk.php`, upon visiting which the string “hacked” is displayed on the web page indicating that arbitrary php can be executed. Therefore, Pentest Monkey’s script can be used to obtain a reverse shell.

Using the script and then executing the PHP code at `http://192.168.59.76:8000/storage/originals/ec/a3/cyberpunk-shell.php` grants a shell with the user `www-data`.

---

## Privilege Escalation

Looking at the SUID binaries using `find / -perm -4000 2>/dev/null` there is an SUID binary for `php7.2` which does have a root shell escalation associated with it. Using `php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"` to launch a shell with effective uid as root, the root flag and the local flag can be read. A new user with full sudo permissions can also be added by modifying the `/etc/passwd` and `/etc/sudoers` files.

---
