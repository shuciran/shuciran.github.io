---
description: >-
  Crackmapexec, tool to enumerate multiple protocols such as smb, rdp and ldap
title: Crackmapexec                   # Add title here
date: 2022-11-14 08:00:00 -0600                           # Change the date to match completion date
categories: [02 Lateral Movement, Crackmapexec]                     # Change Templates to Writeup
tags: [crackmapexec]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### SMB
```bash
crackmapexec smb 10.10.11.158 -u users -p creds
```
Example:
[[StreamIO#^11d8df]]

### LDAP
```bash
crackmapexec ldap 10.10.11.158 -u users -p creds --continue-on-success
```
[[StreamIO#^676765]]

### WINRM
```bash
crackmapexec winrm 192.168.143.59 -u Allison -p 'RockYou!' -d offsec.local
```
