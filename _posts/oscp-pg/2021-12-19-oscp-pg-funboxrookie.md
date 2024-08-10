---
title: OffSec PG - FunBoxRookie
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.80.107`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.80.107`

OS Detection &rarr;  `OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 21       | FTP         | ProFTPD 1.3.5e                                               |
| 22       | SSH         | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.29 ((Ubuntu))                               |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.80.107 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- index.html
- robots.txt

Robots txt file contains entry of `/logs/`. However, this is not reachable.

---

## Exploitation

The ftp actually allows anonymous login despite the nmap service scan not listing it. Looking at the contents, there are a bunch of zip files with different user names. Cracking these files with fcrackzip against the rockyou.txt list of passwords, the following zip files were successfully cracked &rarr;

- cathrine.zip - `catwoman`
- homer.zip - `catwoman`
- tom.zip - `iubire`

Unzipping all these with the respective passwords give an `id_rsa` file for ssh login. All the files are the same, therefore there is only 1 user. Therefore, trying all usernames, the one that works and grants a shell is the user `tom`.

This gives the user flag. Navigation and other actions seem limited, therefore, checked shell and this revealed shell as `/bin/rbash`. This is a restricted shell. Used python to spawn a bash instance to escape this.

---

## Privilege Escalation

Looking at the user directory, there is a mysql history file. The key entry here is that of &rarr; `insert\040into\040support\040(tom,\040xx11yy22!);`. This indicates additions of the user `tom` into the table `support`.

Checking to see if this is the password for the user `tom` by using `sudo -l`, it indeed works out and shows that `tom` can perform all operations under `sudo` without a password. Therefore, using `sudo su` to get root shell, the root flag can be retrieved.

---
