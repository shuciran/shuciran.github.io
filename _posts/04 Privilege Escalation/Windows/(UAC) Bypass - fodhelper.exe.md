UAC can be bypassed in various ways. In this first example, we will demonstrate a technique that allows an administrator user to bypass UAC by silently elevating our integrity level from medium to high.

Most of the publicly known UAC bypass techniques target a specific operating system version. In this case, the target is our lab client running Windows 10 build 1709. We will leverage an interesting UAC bypass based on fodhelper.exe, a Microsoft support application responsible for managing language changes in the operating system. Specifically, this application is launched whenever a local user selects the "Manage optional features" option in the "Apps & features" Windows Settings screen.

![Figure 2: Managing optional features](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/76ed7adc5e1a3fae344ad6985cabdd3e-privilege_escalation_17.png)

As we will soon demonstrate, the fodhelper.exe binary runs as _high integrity_ on Windows 10 1709. We can leverage this to bypass UAC because of the way fodhelper interacts with the Windows Registry. More specifically, it interacts with registry keys that can be modified without administrative privileges. We will attempt to find and modify these registry keys in order to run a command of our choosing with _high integrity_.

The Windows Registry is a hierarchical database that stores critical information for the operating system and for applications that choose to use it. The registry stores settings, options, and other miscellaneous information in a hierarchical tree structure of hives, keys, sub-keys, and values.

We'll begin our analysis by running the C:\\Windows\\System32\\fodhelper.exe binary, which presents the _Manage Optional Features_ settings pane:

![Figure 3: Running fodhelper.exe from the command line](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/7b8a92a9aee549d3ee92c580ad586d72-privilege_escalation_01a.png)

Figure 3: Running fodhelper.exe from the command line

In order to gather detailed information regarding the fodhelper integrity level and the permissions required to run this process, we will inspect its _application manifest_. The application manifest is an XML file containing information that lets the operating system know how to handle the program when it is started. We'll inspect the manifest with the sigcheck utility from Sysinternals, passing the -a argument to obtain extended information and -m to dump the manifest.

```
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

We can use _Process Monitor_ from the Sysinternals suite to gather more information about this tool as it executes.

Process Monitor is an invaluable tool when our goal is to understand how a specific process interacts with the file system and the Windows registry. It's an excellent tool for identifying flaws such as Registry hijacking, DLL hijacking, and more.

After starting procmon.exe, we'll run fodhelper.exe again and set filters to specifically focus on the activities performed by our target process.

![Figure 4: Procmon filter by Process Name](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/aa2c3bbd6aaab101043fcc3f163e3d70-privilege_escalation_13.png)

This filter significantly reduced the output but for this specific vulnerability, we are only interested in how this application interacts with the registry keys that can be modified by the current user. To narrow our results, we will adjust the filter with a search for "Reg", which Procmon uses to mark registry operations.

![Figure 5: Procmon filter by Operation](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/3510f7a331272e693bb9675922cafc35-privilege_escalation_03.png)

Once our new filter has been added, we should only see results for registry operations. Following screenshot shows Process Monitor reduced output as a result of our two filters.

![Figure 6: Procmon filter by Process Name and Operation result](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/31583e231eea24ae00f854c055d01055-privilege_escalation_14.png)

These are more manageable results but we want to further narrow our focus. Specifically, we want to see if the fodhelper application is attempting to access registry entries that do not exist. If this is the case and the permissions of these registry keys allow it, we may be able to tamper with those entries and potentially interfere with actions the targeted high-integrity process is attempting to perform.

To again narrow our search, we will rerun the application and add a "Result" filter for "NAME NOT FOUND", an error message that indicates that the application is attempting to access a registry entry that does not exist.

![Figure 7: Procmon filter by Result](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/285e522950b32b808c0bafb38fb84c88-privilege_escalation_04.png)

The output reveals that fodhelper.exe does, in fact, generate the "NAME NOT FOUND" error, an indicator of a potentially exploitable registry entry.

![Figure 8: Procmon filter by Result result](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/9e882842e94390586a5d9664c9131471-privilege_escalation_15.png)

However, since we cannot arbitrarily modify registry entries in every hive, we need to focus on the registry hive we can control. In this case, we will focus on the HKEY_CURRENT_USER (_HKCU_) hive, which we, the current user, have read and write access to:

![Figure 9: Procmon filter by Path](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/7054a37c909e5f7e87ecbe8d2f254fbe-privilege_escalation_12.png)

Applying this additional filter produces the following results:

![Figure 10: fodhelper.exe looking for command value](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/6a98e8fad641bfda35f18582a83d4e90-privilege_escalation_05.png)

Figure 10: fodhelper.exe looking for command value

According to this output, we see something rather interesting. The fodhelper.exe application attempts to query the 
```c
HKCU:\Software\Classes\ms-settings\shell\open\command
```
registry key, which does not appear to exist.

In order to better understand why this is happening and what exactly this registry key is used for, we'll modify our check under the _Path_ and look specifically for any access to entries that contain ms-settings\\shell\\open\\command. If the process can successfully access that key in some other hive, the results will provide us with more clues.

![Figure 11: Shell\\open\\command execution path](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/e801623595c6c5ec72149ce318841765-privilege_escalation_18.png)

This output contains an interesting result. When fodhelper does not find the 
```c
ms-settings\shell\open\command
```
registry key in HKCU, it immediately tries to access the same key in the HKEY_CLASSES_ROOT (HKCR) hive. Since that entry does exist, the access is successful.

If we search for 
```c
HKCR:ms-settings\shell\open\command
```
in the registry, we find a valid entry:

![Figure 12: DelegateExecute registry entry](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/3132e2091a599cc0e9184f40ef7c44bf-privilege_escalation_19.png)

Based on this observation, and after searching the MSDN documentation for this registry key format (_application-name\\shell\\open_), we can infer that fodhelper is opening a section of the Windows Settings application (likely the Manage Optional Features presented to the user when fodhelper is launched) through the _ms-settings:_ application protocol. An application protocol on Windows defines the executable to launch when a particular URL is used by a program. These URL-Application mappings can be defined through Registry entries similar to the _ms-setting_ key we found in HKCR. In this particular case, the application protocol schema for ms-settings passes the execution to a COM object rather than to a program. This can be done by setting the _DelegateExecute_ key value to a specific COM class ID as detailed in the MSDN documentation.

This is definitely interesting because fodhelper tries to access the _ms-setting_ registry key within the HKCU hive first. Previous results from Process Monitor clearly showed that this key does not exist in HKCU, but we should have the necessary permissions to create it. This could allow us to hijack the execution through a properly formatted protocol handler. Let's try to add this key with the _REG_ utility:

```
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command
The operation completed successfully.

C:\Users\admin>
```

Once we have added the registry key, we will clear all the results from Process Monitor (using the icon highlighted below), restart fodhelper.exe, and monitor the process activity:

![Figure 13: Clearing the output of Process Monitor](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/d858977c1cf11640f572c129c57dd73c-privilege_escalation_06.png)

Please note that clearing the output display does NOT clear the filters we created. They are saved and we do not need to recreate them.

![Figure 14: Getting the output of Process Monitor again](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/1c4a6e99145fcd79e963b19a2f115bc9-privilege_escalation_16.png)

The figure above shows that, this time, fodhelper.exe attempts to query a value (DelegateExecute) stored in our newly-created command key. This did not happen before we created our fake application protocol key. However, since we do not want to hijack the execution through a COM object, we'll add a DelegateExecute entry, leaving its value empty. Our hope is that when fodhelper discovers this empty value, it will follow the MSDN specifications for application protocols and will look for a program to launch specified in the Shell\\Open\\command\\Default key entry.

We will use REG ADD with the /v argument to specify the value name and /t to specify the type:

```
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ
The operation completed successfully.
```

In order to verify that fodhelper successfully accesses the DelegateExecute entry we have just added, we will remove the "NAME NOT FOUND" filter and replace it with "SUCCESS" to show only successful operations and restart the process again:

![Figure 15: fodhelper.exe inspecting the (Default) value under the command registry key](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/e596ab13ed92e0d7e020a46d5bea9fc9-privilege_escalation_07.png)

As expected, fodhelper finds the new DelegateExecute entry we added, but since its value is empty, it also looks for the _(Default)_ entry value of the Shell\\open\\command registry key. The _(Default)_ entry value is created as null automatically when adding any registry key. We will follow the application protocol specifications and replace the empty _(Default)_ value with an executable of our choice, cmd.exe. This should force fodhelper to handle the _ms-settings:_ protocol with our own executable!

In order to test this theory, we'll set our new registry value. We'll also specify the new registry value with /d "cmd.exe" and /f to add the value silently.

```
C:\Users\admin> REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /d "cmd.exe" /f
The operation completed successfully.
```

After setting the value and running fodhelper.exe once again, we are presented with a command shell:

![Figure 16: Spawning a high privileged cmd.exe via fodhelper.exe](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/7e4fe83b49e9c574048e5a00bdf46a57-privilege_escalation_08a.png)

The output of the whoami /groups command indicates that this is a high-integrity command shell. Next, we'll attempt to change the admin password to see if we can successfully bypass UAC:

```
C:\Windows\system32> net user admin Ev!lpass
The command completed successfully.
```

The password change is successful and we have successfully bypassed UAC!

This attack not only demonstrates a terrific UAC bypass, but also reveals a process that we could use to discover similar bypasses.