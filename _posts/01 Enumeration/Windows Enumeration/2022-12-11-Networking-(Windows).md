---
description: >-
  Enumerate Network related services and files on Windows Operating Systems.
title: Networking (Windows)                 # Add title here
date: 2022-12-11 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration, networking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### System Network Configuration
```c
ipconfig /all
netstat -abno
arp -a
```

#### Enumerating Running Processes and Services
Keep in mind that this output does not list processes run by privileged users. On Windows-based systems, we'll need high privileges to gather this information, which makes the process more difficult.
```c
tasklist /SVC

Image Name                     PID Services
========================= ======== ============================================
...
lsass.exe                      564 KeyIso, Netlogon, 
```

#### Networking Routing Tables
```c
route print
```

#### View the active network connections
```c
netstat -ano
```

#### Firewall rules
```c
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all
```

#### Scheduled Tasks
```
schtasks /query /fo LIST /v
```

#### Active Endpoints
```cmd
for /L %i in (1,1,255) do @ping -n 1 -w 200 10.5.5.%i > nul && echo 10.5.5.%i is up.
```

# Additional Tools
#### [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/)
#### [Process Hacker](https://processhacker.sourceforge.io/)
#### [GhostPack Seatbelt](https://github.com/GhostPack/Seatbelt)
