---
description: >-
  Kernel Vulnerabilities via Drivers
title: Kernel Vulnerabilities via Drivers             # Add title here
date: 2022-11-11 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - Kernel Vulnerabilities via Drivers]                     # Change Templates to Writeup
tags: [kernel, software vulnerability, windows privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
When attempting to exploit system-level software (such as drivers or the kernel itself), we must pay careful attention to several factors including the target's operating system, version, and architecture. Failure to accurately identify these factors can trigger a Blue Screen of Death (BSOD) while running the exploit. This can adversely affect the client's production system and deny us access to a potentially valuable target.

### Identify Operating System

Considering the level of care we must take, in the following example we will first determine the version and architecture of the target operating system.

```powershell
C:\> systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
OS Name:                   Microsoft Windows 7 Professional
OS Version:                6.1.7601 Service Pack 1 Build 7601
System Type:               X86-based PC
```

The output of the command reveals that our target is running Windows 7 SP1 on an x86 processor.

At this point, we could attempt to locate a native kernel vulnerability for Windows 7 SP1 x86 and use it to elevate our privileges. However, third-party driver exploits are more common. As such, we should always attempt to investigate this attack surface first before resorting to more difficult attacks.

### Identify Vulnerable Drivers

To do this, we'll first enumerate the drivers that are installed on the system:

```powershell
C:\Users\student\Desktop>driverquery /v

Module Name  Display Name           Description            Driver Type   Start Mode State      Status     Accept Stop Accept Pause Paged Pool Code(bytes) BSS(byLink) Date              Path                                             Init(bytes
============ ====================== ====================== ============= ========== ========== ========== =========== ============ ========== ========== ============================ ================================================ ========
==

ACPI         Microsoft ACPI Driver  Microsoft ACPI Driver  Kernel        Boot    Running    OK         TRUE        FALSE        77,824     143,360    0 11/20/2010 12:37:52 AM C:\Windows\system32\drivers\ACPI.sys             8,192

...

USBPcap      USBPcap Capture Servic USBPcap Capture Servic Kernel        Manual     Stopped    OK         FALSE       FALSE        7,040      9,600      0 10/2/2015 2:08:15 AM   C:\Windows\system32\DRIVERS\USBPcap.sys          2,176

...
```

The output primarily consists of typical Microsoft-installed drivers and a very limited number of third party drivers such as USBPcap. It's important to note that even though this driver is marked as stopped, we may still be able to interact with it, as it is still loaded in the kernel memory space.

Since Microsoft-installed drivers have a rather rigorous patch cycle, third-party drivers often present a more tempting attack surface. For example, let's search for USBPcap in the Exploit Database:

```bash
kali@kali:~# searchsploit USBPcap
--------------------------------------- ----------------------------------------
 Exploit Title                         |  Path
                                       | (/usr/share/exploitdb/)
--------------------------------------- ----------------------------------------
USBPcap 1.1.0.0 (WireShark 2.2.5) - Lo | exploits/windows/local/41542.c
--------------------------------------- ----------------------------------------
```

The output reports that there is one exploit available for USBPcap. As shown in the image below, this particular exploit targets our operating system version, patch level, and architecture. However, it depends on a particular version of the driver, namely USBPcap version 1.1.0.0, which is installed along with Wireshark 2.2.5.

```powershell
Exploit Title    - USBPcap Null Pointer Dereference Privilege Escalation
Date             - 07th March 2017
Discovered by    - Parvez Anwar (@parvezghh)
Vendor Homepage  - http://desowin.org/usbpcap/ 
Tested Version   - 1.1.0.0  (USB Packet cap for Windows bundled with WireShark 2.2.5)
Driver Version   - 1.1.0.0 - USBPcap.sys
Tested on OS     - 32bit Windows 7 SP1 
CVE ID           - CVE-2017-6178
Vendor fix url   - not yet
Fixed Version    - 0day
Fixed driver ver - 0day
...
```

Let's take a look at our target system to see if that particular version of the driver is installed.

To begin, we will list the contents of the Program Files directory, in search of the USBPcap directory:

```powershell
C:\Users\n00b> cd "C:\Program Files"

C:\Program Files> dir
...
08/13/2015  04:04 PM    <DIR>          MSBuild
07/14/2009  06:52 AM    <DIR>          Reference Assemblies
01/24/2018  02:30 AM    <DIR>          USBPcap
12/22/2017  04:11 PM    <DIR>          VMware
04/12/2011  04:16 AM    <DIR>          Windows Defender
...
```

As we can see, there is a USBPcap directory in C\\Program Files. However, keep in mind that the driver directory is often found under C:\\Windows\\System32\\DRIVERS. Let's inspect the contents of USBPcap\\USBPcap.inf to learn more about the driver version:

```powershell
C:\Program Files\USBPcap> type USBPcap.inf
[Version]
Signature           = "$WINDOWS NT$"
Class               = USB
ClassGuid           = {36FC9E60-C465-11CF-8056-444553540000}
DriverPackageType   = ClassFilter
Provider            = %PROVIDER%
CatalogFile.NTx86   = USBPcapx86.cat
CatalogFile.NTamd64 = USBPcapamd64.cat
DriverVer=10/02/2015,1.1.0.0

[DestinationDirs]
DefaultDestDir = 12
...
```

Based on the version information, our driver should be vulnerable. Before we try to exploit it, we first have to compile the exploit since it's written in C.

### Compiling C/C++ Code on Windows

The vast majority of exploits targeting kernel-level vulnerabilities (including the one we have selected) are written in a low-level programming language such as C or C++ and therefore require compilation. Ideally, we would compile the code on the platform version it is intended to run on. In those cases, we would simply create a virtual machine that matches our target and compile the code there. However, we can also cross-compile the code on an operating system entirely different from the one we are targeting. For example, we could compile a Windows binary on our Kali system.

For the purposes of this module however, we will use _Mingw-w64_, which provides us with the GCC compiler on Windows.

Since our Windows client has Mingw-w64 pre-installed, we can run the mingw-w64.bat script that sets up the _PATH_ environment variable for the gcc executable. Once the script is finished, we can execute gcc.exe to confirm that everything is working properly:

```powershell
C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1> mingw-w64.bat

C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1>echo off
Microsoft Windows [Version 10.0.10240]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\> gcc
gcc: fatal error: no input files
compilation terminated.

C:\> gcc --help
Usage: gcc [options] file...
Options:
  -pass-exit-codes         Exit with highest error code from a phase.
  --help                   Display this information.
  --target-help            Display target specific command line options.
  --help={common|optimizers|params|target|warnings|[^]{joined|separate|undocumented}}[
                           Display specific types of command line options.
  (Use '-v --help' to display command line options of sub-processes).
  --version                Display compiler version information.
...
```

Good. The compiler seems to be working. Now let's transfer the exploit code to our Windows client and attempt to compile it. Since the author did not mention any particular compilation options, we will try to run gcc without any arguments other than specifying the output file name with -o.

Despite two warning messages, the exploit compiled successfully and gcc created the exploit.exe executable. If the process had generated an error message, the compilation would have aborted and we would have to attempt to fix the exploit code and recompile it.

Now that we have compiled our exploit, we can transfer it to our target machine and attempt to run it. In order to determine if our privilege escalation was successful, we can use the whoami command before and after running our exploit.