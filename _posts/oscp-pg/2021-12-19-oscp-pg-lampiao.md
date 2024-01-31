---
title: OffSec PG - Lampiao
date: 2021-12-19 12:00:00 +0500
categories: [Lab Practice Notes, OffSec Proving Grounds]
tags: [oscp,lab,offsec-proving-grounds]
image:
  path: /assets/img/covers/pg-cover.jpeg
  alt: PG Artwork
---

## Enumeration

Machine IP &rarr; `192.168.56.48`

### Network Scan

Nmap scan &rarr; `nmap -A -Pn -p- -T4 -o nmap.txt 192.168.56.48`

OS Detection &rarr;  `OS: Linux; CPE: cpe:/o:linux:linux_kernel`

| **Port** | **Service** | **Other details (if any)**                                      |
| -------- | ----------- | --------------------------------------------------------------- |
| 22       | SSH         | OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0) |
| 80       | HTTP?       | \-                                                              |
| 1898     | HTTP        | Apache httpd 2.4.7 ((Ubuntu)) & Drupal 7                        |

### Web Scan

Robots.txt check by nmap listed a ton of directories and files for all user agents. Therefore, listed the entire file using curl, which resulted in the following interesting entries &rarr;

- admin/
- user/register/
- user/password/
- user/login/
- ?q=admin
- ?q=comment/reply/
- ?q=user/register/
- ?q=user/password/

GoBuster scan &rarr; `gobuster dir -u http://192.168.56.48 -w /home/tanq/installations/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php`

Directories/files listed were common with robots.txt finding and the important ones were &rarr;

- install.php
- update.php
- cron.php
- xmlrpc.php
- index.php
- misc/
- scripts/
- includes/
- sites/
- modules/
- themes/

---

## Exploitation

Nothing in the directory and file brute force was particularly interesting. The /CHANGELOG.txt file can be looked at to identify the exact version of software running on the server. This revealed the latest version to be Drupal 7.54.

A particular exploit from Google-fu is the drupalgeddon2. For the version 7.5X, the PoC for the `?q=user/password` would work well. The exploit is basically a lack of input validation in Drupal 7 Form API. This attack targets AJAX requests composed of Drupal Form API’s renderable arrays, which are used to render a requested page through Drupal’s theming system.

Renderable arrays are implemented by an associative array and pass key-value pairs as function arguments or form data in order to render markup and UI elements in a meaningful way. Markup element properties have keys prefixed with the ‘#’ character. These parameters were not sanitized. The exploit code had to target the rendering phase of either a page load or AJAX request with malicious code passed to one of the Form API executable functions. The 4 possibilities were &rarr;

1. [#post_render]
2. [#pre_render]
3. [#access_callback]
4. [#lazy_builder]

To exploit, the following set of 2 commands can be executed to get the result of a bash command on the server &rarr;

```
## 1st command - payload is "which nc" to check if nc is there on the system.
url_pre="http://192.168.56.48:1898/?q=user/password&name"
url_post="$url_pre\[%23post_render\]\[\]=passthru&name\[%23type\]=markup&name\[%23markup\]="
final_url="$url_post=which+nc"
form_data=$( curl -k -s $(echo final_url) --data "form_id=user_pass&_triggering_element_name=name")
form_build=$(echo form_data | grep form_build_id)
form_build_id=$(echo form_build | sed -E 's/.*name="form_build_id" value="(.*)".*/\1/' )

## 2nd command, which returns the result of the above payload.
curl -k -i "http://192.168.56.48:1898/?q=file/ajax/name/%23value/${form_build_id}" \
    --data "form_build_id=${form_build_id}"
```

This gave an indication that nc was present on the system, therefore a reverse shell could be generated for it.

However, an nc shell did not work. It is known that there is php on the system. Therefore, after checking the presence of wget, the php reverse shell from pentest monkey is sent to the server. With a local netcat listener and navigating to the reverse shell path, a shell with the user `www-data` is obtained.

---

## Privilege Escalation

### User

Looking at `/etc/passwd`, a number of users are determined. SetUID binaries do not have any interesting information. The `/etc/passwd` contains the hashed password for the `root` user. This was sent to john for a cracking attempt, which did not reveal anything.

Listing the `/home` directory shows the presence of a user `tiago`. Google-fu for default drupal config files reveals the location as `settings.php` file inside the `sites/default/` directory. This reveals the mysql database connection credentials as `drupaluser:Virgulino`. Trying the password for the user `tiago` successfully logs in. This gives the user flag.

### Root

The `tiago` user is not allowed to run sudo either. Therfore, looking at other information and running the linux enumeration script. Looking at the kernel version, which is 4.4.0, Google-fu points to the presence of the Dirty Cow exploit.

Therefore, looking at PoCs from [Dirty Cow PoCs](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs), choosing the `cowroot.c` payload to get the root shell. Therefore, compiling for 32 bit version to match target. Compiled it with gcc and sent the binary to the target machine via wget.

Executing this gives the root shell and thus the root flag.

---
