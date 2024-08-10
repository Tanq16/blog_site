---
title: OffSec PG - Inclusiveness
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.80.14`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.80.14`

OS Detection &rarr;  `OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 21       | FTP         | vsftpd 3.0.3 &rarr; *Anonymous FTP allowed*         |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.80.14 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- robots.txt
- seo.html
- javascript/
- manual/

Looking at the robots.txt, it says only search engines are allowed to access it. Therefore, changing the User-Agent to `GoogleBot` in burp allows bypassing this restriction. This gives the directory `/secret_information/`.

---

## Exploitation

### LFI

The `/secret_information/` directory consists of an introduction to DNS Zone transfer attacks and links to display it in English or Spanish. The links are of the form `..?lang=language.php`. Therefore, attempting LFI here allows to print contents of the `/etc/passwd` file. This gives enumeration of the users &rarr; `tom` and `root`.

The anonymous ftp login shows that the `pub` directory of the FTP service is world writeable. Therefore, it is a good place for landing payloads. To get the exact path of the location, the configuration file must be read. From the service enumeration, the version is known to be vsftpd 3.0.3. The default config for this is at `/etc/vsftpd.conf`.

Using LFI to print this shows the following &rarr;

```
anon_root=/var/ftp/
write_enable=YES
```

### Reverse shell from Anonymous write-enabled FTP and LFI

Using the FTP to upload a reverse shell in PHP and then using LFI to navigate to the payload using the path found in the config file grants the shell as user `www-data`. The payload used is pentest monkey's PHP reverse shell.

Enumerating the setuid binaries, an interesting find was the presence of `/home/tom/rootshell`, which indicates getting privilege of user `tom` is the step required to get root on the machine.

---

## Privilege Escalation

### User

The home directory of the user `tom` is readable by `www-data`. Therefore, visiting it grants access to the code of the rootshell binary found above. This also gives the user flag.

The code of the rootshell binary uses `FILE* f = popen("whoami", "r");`. This does not use an exact path, therefore, the PATH variable can be abused to trick the program into evaluating the username as `tom`. Therefore, creating a new directory in `/tmp` and an executable `whoami` under it that prints tom allows adding this to the current PATH.

```
echo '#!/bin/bash' > whoami
echo 'echo tom' >> whoami
chmod +x whoami
export PATH=/tmp/testdirectory
```

This allows for execution of the rootshell binary, which evaluates all checks to true and grants the root shell, thereby the root flag.

---
