---
description: >-
  Powerview Modules
title: PowerView Modules             # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, AD - Enumeration]                     # Change Templates to Writeup
tags: [active directory, enumeration, powerview, get-netloggedon, get-netsession, get-domainuser]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Calling an operating system API from PowerShell is not completely straightforward. Fortunately, other researchers have presented a technique that simplifies the process and also helps avoid endpoint security detection. The most common solution is the use of [PowerView](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1), a PowerShell script which is a part of the PowerShell Empire framework.

To use it we must download it and first import it:

```powershell
PS C:\Tools\active_directory> Import-Module .\PowerView.ps1
```

### Currently Logged on Users
We can enumerate logged-in users with Get-NetLoggedon along with the -ComputerName option to specify the target workstation or server. Since in this case we are targeting the Windows 10 client, we will use -ComputerName client251:
```powershell
PS C:\Tools\active_directory> Get-NetLoggedon -ComputerName client251
```

### Currently Active Session
We can invoke the Get-NetSession function in a similar fashion using the -ComputerName flag. Recall that this function invokes the Win32 API _NetSessionEnum_, which will return all active sessions, in our case from the domain controller.
```powershell
PS C:\Tools\active_directory> Get-NetSession -ComputerName dc01
```

### Get-DomainUser
Extract User domain:
```powershell
$SecPassword = ConvertTo-SecureString 'PasswordForSearching123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('FULCRUM\LDAP', $SecPassword)
Get-DomainUser -Credential $Cred
```
Examples:
[Fulcrum](https://shuciran.github.io/posts/Fulcrum/#fnref:powerview-getdomain-user)