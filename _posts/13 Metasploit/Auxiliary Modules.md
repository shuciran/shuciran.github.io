The Metasploit Framework includes hundreds of auxiliary modules that provide functionality such as protocol enumeration, port scanning, fuzzing, sniffing, and more. The modules all follow a common slash-delimited hierarchical syntax (_module type/os, vendor, app, or protocol/module name_), which makes it easy to explore and use the modules. Auxiliary modules are useful for many tasks, including information gathering (under the gather/ hierarchy), scanning and enumeration of various services (under the scanner/ hierarchy), and so on.

There are too many to cover here, but we will demonstrate the syntax and operation of some of the most common auxiliary modules. As an exercise, explore some other auxiliary modules as they are an invaluable part of the Metasploit Framework.

To list all auxiliary modules, we run the show auxiliary command. This will present a very long list of all auxiliary modules as shown in the truncated output below:

```
msf6 auxiliary(scanner/portscan/tcp) > show auxiliary

Auxiliary
=========

   Name                                 Rank    Description
   ----                                 ----    -----------
   ...
   941   scanner/smb/impacket/dcomexec                                  2018-03-19       normal  No     DCOM Exec
   942   scanner/smb/impacket/secretsdump                                                normal  No     DCOM Exec
   943   scanner/smb/impacket/wmiexec                                   2018-03-19       normal  No     WMI Exec
   944   scanner/smb/pipe_auditor                                                        normal  No     SMB Session Pipe Auditor
   945   scanner/smb/pipe_dcerpc_auditor                                                 normal  No     SMB Session Pipe DCERPC Auditor
   946   scanner/smb/psexec_loggedin_users                                               normal  No     Microsoft Windows Authenticated Logged In Users Enumeration
   947   scanner/smb/smb_enum_gpp                                                        normal  No     SMB Group Policy Preference Saved Passwords Enumeration
   948   scanner/smb/smb_enumshares                                                      normal  No     SMB Share Enumeration
   949   scanner/smb/smb_enumusers                                                       normal  No     SMB User Enumeration (SAM EnumUsers)
   950   scanner/smb/smb_enumusers_domain                                                normal  No     SMB Domain User Enumeration
   951   scanner/smb/smb_login                                                           normal  No     SMB Login Check Scanner
   952   scanner/smb/smb_lookupsid                                                       normal  No     SMB SID User Enumeration (LookupSid)
   953   scanner/smb/smb_ms17_010                                                        normal  No     MS17-010 SMB RCE Detection
   954   scanner/smb/smb_uninit_cred                                                     normal  Yes    Samba _netr_ServerPasswordSet Uninitialized Credential State
   955   scanner/smb/smb_version                                                         normal  No     SMB Version Detection
   ...
```

We can use search to reduce this considerable output, filtering by app, type, platform, and more. For example, we can search for SMB auxiliary modules with search type:auxiliary name:smb as shown in the following listing.

```
msf6 auxiliary(scanner/portscan/tcp) > search -h
Usage: search [<options>] [<keywords>:<value>]

Prepending a value with '-' will exclude any matching results.
If no options or keywords are provided, cached results are displayed.

OPTIONS:
  -h                Show this help information
  -o <file>         Send output to a file in csv format
  -S <string>       Regex pattern used to filter search results
  -u                Use module if there is one result

Keywords:
  aka         :  Modules with a matching AKA (also-known-as) name
  author      :  Modules written by this author
  arch        :  Modules affecting this architecture
  bid         :  Modules with a matching Bugtraq ID
  cve         :  Modules with a matching CVE ID
  ...
  target      :  Modules affecting this target
  type        :  Modules of a specific type (exploit, payload, auxiliary, encoder, evasion, post, or nop)

Examples:
  search cve:2009 type:exploit
  search cve:2009 type:exploit platform:-linux

  
msf6 auxiliary(scanner/portscan/tcp) > search type:auxiliary name:smb

Matching Modules
================

   #   Name                                                        Disclosure Date  Rank    Check  Description
   -   ----                                                        ---------------  ----    -----  -----------
   0   auxiliary/admin/oracle/ora_ntlm_stealer                     2009-04-07       normal  No     Oracle SMB Relay Code Execution
   1   auxiliary/admin/smb/check_dir_file                                           normal  No     SMB Scanner Check File/Directory Utility
   2   auxiliary/admin/smb/delete_file                                              normal  No     SMB File Delete Utility
   3   auxiliary/admin/smb/download_file                                            normal  No     SMB File Download Utility
   4   auxiliary/admin/smb/list_directory                                           normal  No     SMB Directory Listing Utility
   ...
```

After invoking a module with use, we can request more info about it as follows:

```
msf6 auxiliary(scanner/portscan/tcp) > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > info

       Name: SMB Version Detection
     Module: auxiliary/scanner/smb/smb_version
    License: Metasploit Framework License (BSD)
       Rank: Normal

Provided by:
  hdm <x@hdm.io>
  Spencer McIntyre
  Christophe De La Fuente

Check supported:
  No

Basic options:
  Name     Current Setting  Required  Description
  ----     ---------------  --------  -----------
  RHOSTS                    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  THREADS  1                yes       The number of concurrent threads (max one per host)

Description:
  Fingerprint and display version information about SMB servers. 
  Protocol information and host operating system (if available) will 
  be reported. Host operating system detection requires the remote 
  server to support version 1 of the SMB protocol. Compression and 
  encryption capability negotiation is only present in version 3.1.1.
```

The module description output by info tells us that the purpose of the smb2 module is to detect the version of SMB supported by the remote machines. The module's _Basic options_ parameters can be inspected by executing the show options command. For this particular module, we just need to set the IP address of our target, in this case our student Windows 10 machine.

Alternatively, since we have already scanned our Windows 10 machine, we could search the Metasploit database for hosts with TCP port 445 open (services -p 445) and automatically add the results to _RHOSTS_ (--rhosts):

```
msf6 auxiliary(scanner/smb/smb_version) > services -p 445 --rhosts
Services
========

host            port  proto  name          state  info
----            ----  -----  ----          -----  ----
192.168.120.11  445   tcp    microsoft-ds  open   

RHOSTS => 192.168.120.11

msf6 auxiliary(scanner/smb/smb_version) >
```

With the required parameters configured, we can launch the module with run or exploit:

```
msf6 auxiliary(scanner/smb/smb_version) > run

[*] 192.168.120.11:445    - SMB Detected (versions:2, 3) (preferred dialect:SMB 3.1.1) (compression capabilities:LZNT1) (encryption capabilities:AES-128-CCM) (signatures:optional) (guid:{525b6385-a6e7-4052-83d4-cc968af5dcbd}) (authentication domain:DESKTOP-6UTT671)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/smb/smb_version) >
```

Based on the module's output, the remote computer does indeed support SMB version 2. To leverage this, we can use the scanner/smb/smb_login module to attempt a brute force login against the machine. Loading the module and listing the options produces the following output:

```
msf6 auxiliary(scanner/smb/smb_version) > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > options

Module options (auxiliary/scanner/smb/smb_login):

   Name               Current Setting  Required  Description
   ----               ---------------  --------  -----------
   ABORT_ON_LOCKOUT   false            yes       Abort the run when an account lockout is detected
   BLANK_PASSWORDS    false            no        Try blank passwords for all users
   BRUTEFORCE_SPEED   5                yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS       false            no        Try each user/password couple stored in the current database
   DB_ALL_PASS        false            no        Add all passwords in the current database to the list
   DB_ALL_USERS       false            no        Add all users in the current database to the list
   DETECT_ANY_AUTH    false            no        Enable detection of systems accepting any authentication
   DETECT_ANY_DOMAIN  false            no        Detect if domain is required for the specified user
   PASS_FILE                           no        File containing passwords, one per line
   PRESERVE_DOMAINS   true             no        Respect a username that contains a domain name.
   Proxies                             no        A proxy chain of format type:host:port[,type:host:port][...]
   RECORD_GUEST       false            no        Record guest-privileged random logins to the database
   RHOSTS                              yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT              445              yes       The SMB service port (TCP)
   SMBDomain                           no        The Windows domain to use for authentication
   SMBPass                             no        The password for the specified username
   SMBUser                             no        The username to authenticate as
   STOP_ON_SUCCESS    false            yes       Stop guessing when a credential works for a host
   THREADS            10               yes       The number of concurrent threads (max one per host)
   USERPASS_FILE                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS       false            no        Try the username as the password for all users
   USER_FILE                           no        File containing usernames, one per line
   VERBOSE            true             yes       Whether to print output for all attempts
```

The output reveals that this module accepts both _Required_ parameters (like _RHOSTS_) and optional parameters (like _SMBDomain_). However, we notice that _RHOSTS_ is not set, even though we set it while using the previous smb2 module. This is because set defines a parameter only within the scope of the running module. We can instead set a global parameter, which is available across all modules, with setg.

One parameter that we often change for auxiliary modules is _THREADS_. This parameter tells the framework how many threads to initiate when running the module, increasing the concurrency, and the speed. We don't want to go too crazy with this number, but a slight increase will dramatically decrease the run time.

For the sake of this demonstration, let's assume that we have discovered valid domain credentials during our assessment. We would like to determine if these credentials can be reused on others servers that have TCP port 445 open. To make things easier, we will try this approach on our Windows client, beginning with an invalid password.

We'll start by supplying the valid domain name of corp.com, a valid username (Offsec), an _invalid_ password (ABCDEFG123!), and the Windows 10 target's IP address:

```
msf6 auxiliary(scanner/smb/smb_login) > set SMBUser Offsec
SMBUser => Offsec
msf6 auxiliary(scanner/smb/smb_login) > set SMBPass notarealpassword
SMBPass => notarealpassword
msf6 auxiliary(scanner/smb/smb_login) > setg RHOSTS 192.168.120.11
RHOSTS => 192.168.120.11
msf6 auxiliary(scanner/smb/smb_login) > set THREADS 10
THREADS => 10
msf6 auxiliary(scanner/smb/smb_login) > 


msf6 auxiliary(scanner/smb/smb_login) > run

[*] 192.168.120.11:445    - 192.168.120.11:445 - Starting SMB login bruteforce
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\Offsec:notarealpassword',
[*] 192.168.120.11:445    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Since we knew that the password we supplied was invalid, the login failed as expected. Now, let's try to supply a valid password and re-run the module.

```
msf6 auxiliary(scanner/smb/smb_login) > set SMBPass Qwerty09!
SMBPass => Qwerty09!
msf6 auxiliary(scanner/smb/smb_login) > run

[*] 192.168.120.11:445    - 192.168.120.11:445 - Starting SMB login bruteforce
[+] 192.168.120.11:445    - 192.168.120.11:445 - Success: 'WORKSTATION\Offsec:Qwerty09!'
[*] 192.168.120.11:445    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

This time, the authentication succeeded. We can retrieve information regarding successful login attempts from the database with creds.

```
msf6 auxiliary(scanner/smb/smb_login) > creds
Credentials
===========

host            origin          service        public  private    realm  private_type  JtR Format
----            ------          -------        ------  -------    -----  ------------  ----------
192.168.120.11  192.168.120.11  445/tcp (smb)  Offsec  Qwerty09!         Password      
```

Although this run was successful, this method will not scale well. To test a larger user base with a variety of passwords, we could instead use the _USERPASS_FILE_ parameter, which instructs the module to use a file containing users and passwords separated by space, with one pair per line.

```
msf6 auxiliary(scanner/smb/smb_login) > set USERPASS_FILE /home/kali/users.txt
USERPASS_FILE => /home/kali/users.txt
msf6 auxiliary(scanner/smb/smb_login) > run

[*] 192.168.120.11:445    - 192.168.120.11:445 - Starting SMB login bruteforce
[+] 192.168.120.11:445    - 192.168.120.11:445 - Success: 'WORKSTATION\Offsec:Qwerty09!'
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\bob:Qwerty09!',
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\bob:password',
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\alice:Qwerty09!',
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\alice:password',
[-] 192.168.120.11:445    - 192.168.120.11:445 - Failed: 'WORKSTATION\:',
[*] 192.168.120.11:445    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Let's try out another module. In this example, we will try to identify machines listening on TCP port 3389, which indicates they might be accepting Remote Desktop Protocol (RDP) connections. To do this, we will invoke the scanner/rdp/rdp_scanner module.

```
msf6 auxiliary(scanner/smb/smb_login) > use auxiliary/scanner/rdp/rdp_scanner
msf6 auxiliary(scanner/rdp/rdp_scanner) > show options

Module options (auxiliary/scanner/rdp/rdp_scanner):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   DETECT_NLA       true             yes       Detect Network Level Authentication (NLA)
   RDP_CLIENT_IP    192.168.0.100    yes       The client IPv4 address to report during connect
   RDP_CLIENT_NAME  rdesktop         no        The client computer name to report during connect, UNSET = random
   RDP_DOMAIN                        no        The client domain name to report during connect
   RDP_USER                          no        The username to report during connect, UNSET = random
   RHOSTS           192.168.120.11   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT            3389             yes       The target port (TCP)
   THREADS          1                yes       The number of concurrent threads (max one per host)

msf6 auxiliary(scanner/rdp/rdp_scanner) > run

[*] 192.168.120.11:3389   - Detected RDP on 192.168.120.11:3389   (Windows version: 10.0.18362) (Requires NLA: No)
[*] 192.168.120.11:3389   - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

This module successfully detected RDP running on one host and automatically added the results to the database.