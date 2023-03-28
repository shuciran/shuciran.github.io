---
description: >-
  Enumerate Users Information on Windows Operating Systems.
title: User Information                   # Add title here
date: 2022-11-10 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
## Users
#### User Privileges
```admin
whoami /priv
whoami /groups
```
#### Network Privileges
```bash
net user
net group `domain`
net localgroup `local`
net localgroup <Group Name>
net accounts `domain`
net accounts /domain `local`
```
