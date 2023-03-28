### SMBMAP
```bash
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --download Shared\\Documents\\Analytics\\Whatif.omv
```
[[Anubis#^cbbd94]]

### SMBCLIENT
#### Download a Share
```bash
# First connect to it and then run this commands:
smb: \> RECURSE ON
smb: \> PROMPT OFF
smb: \> mget *
```
Examples:
[[Active#^72bf14]]