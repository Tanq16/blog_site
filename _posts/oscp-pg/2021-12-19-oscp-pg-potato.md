---
title: OffSec PG - Potato
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.53.101`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.53.101`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                   |
| -------- | ----------- | ------------------------------------------------------------ |
| 22       | SSH         | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP        | Apache httpd 2.4.41 ((Ubuntu))                               |
| 2112     | FTP         | ProFTPd                                                      |

### Web Scan

GoBuster scan &rarr; `gobuster dir -u http://192.168.53.101 -f -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt`

Directories/files listed &rarr;

- admin/
- admin/index.php

---

## Exploitation

Used ftp to login to the service running on port 2112, which allowed anonymous login.

The files available were `index.php.bak` and `welcome.msg`. The backup of the index page consists of the following code &rarr;

```html
<html>
<head></head>
<body>
<?php
$pass= "potato"; //note Change this password regularly
if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! <br> Go to the <a href="dashboard.php">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! <br> Return to the <a href="index.php">login page</a> <p>";
  }
  exit();
}
?>
  <form action="index.php?login=1" method="POST">
    <h1>Login</h1>
    <label><b>User:</b></label>
    <input type="text" name="username" required>
    <br>
    <label><b>Password:</b></label>
    <input type="password" name="password" required>
    <br>
    <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>
```

This gives an idea about how to bypass the login on `/admin/index.php` page. The `strcmp($_POST['password'], $pass) == 0)` check can be bypassed by changing the parameter `password` into `password[]` i.e., change it to an array compared to string. That would evaluate to true. Further, the username from the code shows that the required user is admin. Doing the check as stated by intercepting in Burp and changing parameters, a dashboard page at `/admin/dashboard.php` is made available.

The dashboard page has a Logs section that can retrieve logs from the system. This could have a directory traversal i.e., LFI vulnerability. Catching the request in burp and changing the log file to `../../../../../../../../etc/passwd` gives the required users from the etc passwd file which have the bash login shell &rarr; `root`, `florianges`, `webadmin`.

The entry of interest is webadmin. The hash is available and seems insecure which can be cracked using John the Ripper. For this, the entry `webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash` is stored in a file `test.pass` and John is run as follows &rarr; `john --wordlist=rockyou.txt test.pass`. This revealed the output `dragon` which is the password for the webadmin user.

Used the password `dragon` for the user webadmin for ssh as follows &rarr; `ssh webadmin@192.168.53.101`. This gives the user flag located in the home directory for the user.

---

## Privilege Escalation

Searched for setuid binaries with command &rarr; `find / -perm -u=s -type f 2>/dev/null`. Without a usable binary, checked allowed executions with sudo as follows &rarr; `sudo -l`. This gives the result that the user is allowed to run all commands under `/bin/nice /notes/*`.

The `/notes/` directory contained scripts for executing `clear` and `id` commands. The `*` here is a wildcard and can thus be used to bypass the strict controls and use directory traversal technique to execute bash in the elevated state.

Therefore, executing the `/bin/nice` command as follows &rarr; `sudo /bin/nice /notes/../bin/bash` gives the root shell, followed by the root flag in the root home directory.

---
