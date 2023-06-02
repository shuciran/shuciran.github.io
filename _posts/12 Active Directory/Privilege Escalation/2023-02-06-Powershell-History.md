---
description: >-
  Powershell History
title: Powershell History              # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Privilege Escalation]                     # Change Templates to Writeup
tags: [active directory, windows privesc, powershell, history, information leakage]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
We can retrieve commands executed on another machine by checking the ConsoleHost_history.txt which is the same as in Linux .bash_history file.
#Note If using this command we need to be on the User's folder since this file is located in the AppData folder:
```bash
PS C:\Users\legacyy> type AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
Examples:
[Timelapse](https://shuciran.github.io/posts/Timelapse/#fnref:impacket-psexec)