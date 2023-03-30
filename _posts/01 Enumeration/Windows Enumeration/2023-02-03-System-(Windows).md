---
description: >-
  Enumerate System Files on Windows Operating Systems.
title: System (Windows)                  # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration, system]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### System information
```c
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" 
systeminfo | findstr /C:"sistema" #Español
```
#### Installed updates
```c
wmic qfe get Caption, Description
```
#### Installed Services
```c
net start
```
#### Installed Apps
```c
wmic product get name,version,vendor
```
#### Installed Executables
```c
dir /s /b *.exe | findstr /v .exe.
```
Useful paths:
```c
c:\windows\system32\drivers\etc\hosts
C:\Windows\win.ini
C:\xampp\php\php.ini
```
#### Enumerating Readable/Writable Files and Directories
Accesschk (Windows Internals)
You need to first move [Accesschk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) to the victim machine.
If this file were to be executed by a privileged user or a service account, we could attempt to overwrite it with a malicious file of our choice, such as a reverse shell, in order to elevate our privileges.
```c
accesschk.exe /accepteula -uws "Everyone" "C:\Program Files"
```
Powershell
We can also accomplish the same goal using PowerShell. This is useful in situations where we may not be able to transfer and execute arbitrary binary files on our target system.
```powershell
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```

#### Enumerating Unmounted Disks
```c
mountvol
```

#### Enumerating Device Drivers and Kernel Modules
On Windows, we can begin our search with the driverquery command. We'll supply the /v argument for verbose output as well as /fo csv to request the output in _CSV_ format.
```powershell
c:\Users\student>powershell

PS C:\Users\student> driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path
```
While this produced a list of loaded drivers, we must take another step to request the version number of each loaded driver.
```powershell
PS C:\Users\student> Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
```
Now that we have a list of all the loaded device drivers along with the respective version numbers, we could search for exploits for these specific drivers.
