---
title: OffSec PG - Vegeta1
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.72.73`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.72.73`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.72.73 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- img/ (301)
- image/ (301)
- admin/ (301)
- manual/ (301)
- server-status (403)
- bulma/ (301)

The robots.txt was checked manually during the scan was running. Robots file reveals a directory called `find_me`. This doesn’t contain any useful information either.

---

## Exploitation

None of the directories were really useful. The login pages inside the `/admin/` directory were empty. The `/bulma/` directory revealed an audio file in the wav format.

Uploading the wav file to an online audio decoder shows that the audio is morse code and the text states the presence of a user `trunks` with a password `u$3r`. This can be used to login to the ssh server running at the target. This gives us the user flag.

---

## Privilege Escalation

Enumerating for `sudo` and setuid binaries on the file system, there was no finding apart from the presence of the setuid binary `su`. Looked at the bash rc and history files. The history file contained the following interesting entries &rarr;

```
perl -le 'print crypt("Password@973","addedsalt")'
echo "Tom:ad7t5uIalqMws:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd
```

Checked the permissions of the `/etc/passwd` file and indeed the user `Trunks` owned the file. This allowed direct manipulation of the file. Therefore, added the above entry to the passwd file. Then logged in as Tom using the password that was encrypted in the above commands.

This gave the root shell and thereby the root flag.

---
