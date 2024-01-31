---
title: OffSec PG - OnSystemShellDread
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.244.130`

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.244.130`

OS Detection &rarr;  `OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 21       | FTP         | vsftpd 3.0.3 &rarr; Anonymous login allowed         |
| 61000    | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |

---

## Exploitation

Looking at the anonymous ftp, there was a directory `.hannah` inside which was an ssh key. Used this key for the user `hannah` on the ssh server grants the shell as `hannah`. This gave the user flag.

---

## Privilege Escalation

Looked at the setuid binaries on the system, `cpulimit` was the interesting one. It can be ised to spawn a shell with elevated privileges using command &rarr; `cpulimit -l 50 -f /bin/bash`. However, with this, the program detects that the program being run has lower privileges, so bash drops the elevated privileges. Usually, bash has a flag `-p`, the purpose of which, as stated in the man page of bash is &rarr;

```
If  the  shell is started with the effective user (group) id not equal to the
real user (group) id, and the -p option is not supplied, the effective user id
is set to the real user id. Otherwise, the effective user id is not reset.
```

Therefore, a privileged shell can be launched as follows &rarr; `cpulimit -l 100 -f -- /bin/sh -p`. This gives the root shell and thus, the root flag.

---
