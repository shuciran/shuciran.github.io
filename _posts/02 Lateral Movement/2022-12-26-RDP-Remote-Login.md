---
description: >-
    How to connect to a remote RDP
title: RDP Remote Login                   # Add title here
date: 2022-12-26 08:00:00 -0600                           # Change the date to match completion date
categories: [02 Lateral Movement, RDP Remote Login]                     # Change Templates to Writeup
tags: [rdp remote login]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
## RDP into a remote machine

### XFREERDP
To access windows via port tcp-3389 into a system:
- /u - user
- /p - password
- /w - weight
- /h - height
- /v - remote machine
```bash
xfreerdp /u:JohnDoe /p:Pwd123! /w:1366 /h:768 /v:192.168.1.100:4489
```
### RDESKTOP
```bash
rdesktop -u Administrator -p studentlab manageengine
```
