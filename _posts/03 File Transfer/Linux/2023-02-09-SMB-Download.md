---
description: >-
  Download a complete File Share with SMB 
title: SMB Download                # Add title here
date: 2023-02-09 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Linux - SMB Download]                     # Change Templates to Writeup
tags: [file transfer, smb download]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### SMBMAP
```bash
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --download Shared\\Documents\\Analytics\\Whatif.omv
```
[[Anubis#^cbbd94]]

### SMBCLIENT
#### Download a Share
```bash
# First connect to it and then run this commands:
smb: \> RECURSE ON
smb: \> PROMPT OFF
smb: \> mget *
```
Examples:
[Active](https://shuciran.github.io/posts/Active/#fnref:smb-download)