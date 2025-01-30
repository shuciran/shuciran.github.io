---
description: >-
  Filethingies echoCTF Machine
title: Filethingies (Advanced)                # Add title here
date: 2025-01-29 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Advanced]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Filethingies.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.223
```

### Content

- Default Credentials (admin:admin)
- File Thingie 2.5.7 - Remote Code Execution (RCE)
- LFI through Web Server running as root 

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
# Nmap 7.94SVN scan initiated Wed Jan 29 01:09:30 2025 as: nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.223
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.223 ()   Status: Up
Host: 10.0.160.223 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   256 91:c0:5d:ff:aa:45:ec:e5:05:91:ca:74:8f:dd:86:ce (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBG7Vc5YmzKar1YnN4aSeb7O39QE2U4WOBLFpBUfNO77vlyKUuCYgZDz2DPlODfkt+cczeRq9OUv4VMx4KPGBlLc=
|   256 c7:b8:08:47:f4:e4:78:08:c2:43:c0:a0:dc:b3:1b:ea (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGVO+CQdBRM4jLLFIjfSAObSUOOBD17zqgy3g8loP1ke
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET
| http-title: File Thingie 2.5.7
|_Requested resource was ft2.php
|_http-server-header: Apache/2.4.61 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Exploitation

First of all, only open port is HTTP 80, there is only a login web page called "File Thingie", same as the machine name:

![](/assets/img/Pasted-image-20250129165347.png)

Searching for exploits, there is one for version [FileThingie 2.5.7](https://www.exploit-db.com/exploits/51436) but it needs credentials to work, using the classic `admin:admin` credentials allows us to access:

![](/assets/img/Pasted-image-20250129170009.png)

Now, by executing the python exploit, it seems like there are three main actions that allows the RCE, first one is the creation of a folder, second the upload of a ZIP file and third is the unzip of it:

![](/assets/img/Pasted-image-20250129170259.png)

Unfortunately, while accessing to this supposedly created page the webshell is not uploaded:

![](/assets/img/Pasted-image-20250129170538.png)

After a lot of modifications to the script I then decided that the best approach is to go with the basics, so I started by fuzzing the web page, just in case there are some hidden folders:

![](/assets/img/Pasted-image-20250129170717.png)

While accessing the `/files` directory and voilá, there is a low vulnerability, Directory Listing, which for this cases is a good approach to try and because we already know what does the exploit does, we then proceed to follow the previous steps manually.

First step is to create a folder called `files` as well as the directory:

![](/assets/img/Pasted-image-20250129171039.png)

Second, upload the file .zip that contains your reverse shell:

![](/assets/img/Pasted-image-20250129171144.png)

Third, unzip it, at first glance it seems like it didn't work, but if you go to the `/files` directory you'll see it:

![](/assets/img/Pasted-image-20250129171315.png)

And we have access as `www-data`:

![](/assets/img/Pasted-image-20250129171419.png)

### Privilege Escalation

Once inside the machine (using netcat) we proceed to lookup for possible ways to escalate privileges, the most interesting part is that there is a service running on port 1337:

![](/assets/img/Pasted-image-20250129173228.png)

While using `wget` it is possible to check if there is a web server running on this port, and yes it is!:

![](/assets/img/Pasted-image-20250129173451.png)

To get a better view of such web server, I ran [Chisel](https://shuciran.github.io/posts/Chisel/) to port forwarding the port to my machine:

![](/assets/img/Pasted-image-20250129175951.png)

At this point I started searching for exploits, but sometimes the answer is more simple, browsing the page I found the Admin panel section on the "Options" menu:
![](/assets/img/Pasted-image-20250129180130.png)

Apparently, you don't need to login if you access via localhost:

![](/assets/img/Pasted-image-20250129181200.png)

Then by using the page, I found a way to read system files, and the user running this webapp is root, so yes, you can read any file by going to the "Shared Files" and then click on ⏏ button:

![](/assets/img/Pasted-image-20250129181320.png)

Finally, you are capable to read `/root` flag and `/etc/shadow`:

![](/assets/img/Pasted-image-20250129181813.png)

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

Default credentials for File Thing App `admin:admin`

### Notes

- Sometimes, the exploits does not work even while modifying them, we need to understand them and run them on our own, because the developers might adjust the third party WebApps for their own use.

### References

- [FileThingie 2.5.7 Vulnerability](https://www.exploit-db.com/exploits/51436)
- [Chisel](https://shuciran.github.io/posts/Chisel/)

