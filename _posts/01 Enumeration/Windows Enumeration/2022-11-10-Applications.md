---
description: >-
  Enumerate Application on Windows Operating Systems.
title: Applications                   # Add title here
date: 2022-11-10 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Enumerating Installed Applications and Patch Levels
```c
wmic product get name, version, vendor
```

#### List system-wide updates with _Win32_QuickFixEngineering (qfe)_
```c
wmic qfe get Caption, Description, HotFixID, InstalledOn
```
