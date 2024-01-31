---
title: OffSec PG - Born2Root
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.225.49`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.225.49`

OS Detection &rarr;  `os_info`

| **Port** | **Service** | **Other details (if any)**                   |
| -------- | ----------- | -------------------------------------------- |
| 22       | SSH         | OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.10 ((Debian))               |
| 111      | RPCBIND     | rpcbind 2-4 (RPC #100000)                    |
| 44532    | \-          | \-                                           |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.225.49 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Robots txt file has two directories &rarr;

- wordpress-blog/
- files/

Directories/files listed &rarr;

- index.html
- robots.txt
- icons/
- files/
- manual/

---

## Exploitation

The `/icons/` directory has a `.txt` file which seems out of place amongst all the other image files. Upon inspection, it contained an RSA private key. Used this to login to the ssh server running on the machine. For ther user, the most obvious one is `martin` given the clear mentions throughout the website. Upon loggin in, the shell asks for a secret password. Entering something random just drops into the shell and gives the user flag.

The program that asked for the secret password was located at the end of the `.bashrc` in `martin`'s home directory. This was `/var/tmp/login.py`. Upon close inspection, the script has an error to always allow access into the shell. The script is as follows &rarr;

```python
#!/usr/bin/python

import os

print("")
print("READY TO ACCESS THE SECRET LAB ? ")
print("")
password = raw_input("secret password : ")

if (password) == "secretsec" or "secretlab" : ## --> Always true
	print("WELCOME ! ")
else:
	print("GET OUT ! ")
	os.system("pkill -u 'martin'")
```

---

## Privilege Escalation

Looking at the `/etc/passwd` file, the users of interest are `root`, `martin`, `hadi` and `jimmy`.

### User 1

Looking at the crontab, there is a job by `jimmy` that runs every 5 minutes as `python /tmp/sekurity.py`. This file does not exist, therefore, can be created and subsequently executed. This can be used to receive a shell as `jimmy` over netcat. The home directory consists of a `networker` binary, which doesn't seem to do anything concrete.

### User 2

Cracking the password for `hadi` was taking too long, therefore a hack was used to grep out all passwords related to "hadi" from the `rockyou.txt` list. This sublist was also set on the cracking task in parallel, which found the password surprisingly quick, resulting in credentials `hadi:hadi123`.

### Root

As the `hadi` user, running `su root` directly gives the root shell and thus the root flag.

---
