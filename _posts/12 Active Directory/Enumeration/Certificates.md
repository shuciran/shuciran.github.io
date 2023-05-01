### Certify.exe
If we are already inside a machine we can execute certify.exe from the [SharpCollections Suite](https://github.com/Flangvik/SharpCollection/tree/master/NetFramework_4.5_Any) this tool will enumerate any vulnerable Certificate by using the following command:
```powershell
.\certify.exe find /vulnerable /currentuser
```
[[Anubis#^32beda]]

### Enumerate certificates
Enumerate certificates with the current user:
```powershell
gci cert:\currentuser\my -verbose


   PSParentPath: Microsoft.PowerShell.Security\Certificate::currentuser\my

Thumbprint                                Subject
----------                                -------
95E99117088DD7F691848064CC95E3F4E03E2C9C
```
[[Anubis#^3fe8c0]]