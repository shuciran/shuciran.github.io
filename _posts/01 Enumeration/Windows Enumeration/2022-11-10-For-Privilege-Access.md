---
description: >-
  Search PrivEsc vectors on Windows Operating Systems.
title: For Privilege Access                   # Add title here
date: 2022-11-10 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
First, on Windows systems, we should check the status of the \_AlwaysInstallElevated registry setting. If this key is enabled (set to _1_) in either _HKEY_CURRENT_USER_ or _HKEY_LOCAL_MACHINE_, any user can run Windows Installer packages with elevated privileges.
We can use reg query to check these settings:
```c
// For current user
C:\> reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer

HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1

// For local machine
c:\>reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer

HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1
```

If this setting is enabled, we could craft an _MSI_ file and run it to elevate our privileges.
