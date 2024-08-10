---
title: OffSec PG - Seppuku
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.244.90`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.141.90`

OS Detection &rarr;  `os_info`

### Table

| **Port** | **Service**   | **Other details (if any)**                     |
| -------- | ------------- | ---------------------------------------------- |
| 21       | FTP           | vsftpd 3.0.3                                   |
| 22       | SSH           | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP          | nginx 1.14.2                                   |
| 139      | NETBIOS-SSN   | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)    |
| 445      | MICROSOFT-DS  | Samba smbd 4.9.5-Debian (workgroup: WORKGROUP) |
| 7080     | SSL/EMPOWERID | LiteSpeed                                      |
| 7601     | HTTP          | Apache httpd 2.4.38 ((Debian))                 |
| 8088     | HTTP          | LiteSpeed httpd                                |

### Web Scan

GoBuster scan &rarr; `/opt/gobuster dir -u http://192.168.141.90 -f -w /opt/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- /index.html
- /icons/
- /b/
- /a/
- /c/
- /t/
- /r/
- /d/
- /e/
- /f/
- /h/
- /w/
- /q/
- /database/
- /production/
- /keys/
- /secret/

The directories `/w/`, `secret` and `keys` contain several interesting files &rarr; 

- ssh private keys within `private` and `private.bak` 
- hostname file with value `seppuku` 
- a wordlist `password.lst` 
- `passwd.bak` and `shadow.bak` 

---

## Exploitation

The wordlist can be used to brute force the ssh login by using hydra as follows &rarr; 

```bash
hydra -l seppuku -P password.lst ssh://192.168.141.90
```

This gives the valid credentials as `seppuku:eeyoree` which can be used to login to the machine. This also gives the user flag within `local.txt` in the home directory.

Other items such as the backup for the password and shadow files were rabbit holes due to incorrectly formatted hashes.

---

## Privilege Escalation

### User

Listing the users in the `/home` directory gives other users as `samurai` and `tanto`. The ssh private keys discovered earlier grant access to user `tanto` via ssh. This leads us to a restricted shell. The `sudo -l` permissions for the user were to only create a symbolic link of the `/root` directory inside `/tmp`. However, this directory would still have the permissions of `root` which means those permissions are still needed to read the root flag.

The user directory also contains a `.passwd` file which contains a password. This password helps login with credentials `samurai:12345685213456!@!@A` for the next user. The `sudo -l` capability for this user is to run the following command &rarr; 

```bash
/../../../../../../home/tanto/.cgi_bin/bin /tmp/*
```

This means that the command tries to execute the file `/home/tanto/.cgi_bin/bin` as a command and the `/tmp/*` as an argument.

### Root

The command `bin` can be replaced with anything such that it will get executed. This can be done by using a shell script by the same name that can be created on the machine using `nano` as well as served via HTTP from attacker host to the `tanto` machine using `wget`. The file should contain the following &rarr; 

```bash
#!/bin/bash

/bin/bash
```

This can then be made world executable by using `chmod 777 bin` and then moved inside the `.cgi_bin` directory. `samurai` can then use the command with sudo to execute this file which would then grant a root shell and thus the root flag.

---
