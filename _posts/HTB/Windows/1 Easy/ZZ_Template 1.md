---
description: >-
  Active HTB Machine
title: Active (Easy)                # Add title here
date: 2023-02-09 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Easy]                     # Change Templates to Writeup
tags: [smb enumeration, smb full replication, gpp decryption, kerberoasting, hashcat, tgs cracking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Active.png                # Add infocard image here for post preview image
---
### Host entries:
```bash
10.10.10.125  
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- 
- 

### Reconnaissance

Initial reconnaissance for TCP ports
```bash

```
Services and Versions running:
```bash

```

### Exploitation


### Root privesc

### Post Exploitation

### Credentials

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### Resources



