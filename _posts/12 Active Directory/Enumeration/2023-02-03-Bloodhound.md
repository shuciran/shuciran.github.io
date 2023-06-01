---
description: >-
  Bloodhound for AD enumeration
title: Bloodhound             # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, AD - Enumeration]                     # Change Templates to Writeup
tags: [active directory, Enumeration, bloodhound, sharphound, bloodhound-python]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Sharphound.exe
First upload [Sharphound](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.exe) to the system and then run the following commands from a folder where you can write as it will download a .zip file:
```powershell
# For SharpHound.ps1 (each line is a command)
PS> Powershell -exec bypass 
PS> Import-module SharpHound.ps1
PS> Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default
# For Sharphound.exe: Upload the file and execute it:
PS> .\SharpHound.exe
```
Examples:

[Streamio](https://shuciran.github.io/posts/Streamio/#fnref:sharphound)
[Intelligence](https://shuciran.github.io/posts/Intelligence/#fnref:bloodhound-python)
[Outdated](https://shuciran.github.io/posts/Outdated/#fnref:sharphound)

### Bloodhound.py
Resources: [bloodhound.py](https://github.com/fox-it/BloodHound.py)
If you have no access to the machine bloodhound.py can be used from the attack machine:
```bash
apt install bloodhound.py
```
Then run it to create the bloodhound files:
```bash
bloodhound-python -c All -u amanda -p Ashare1972 -ns 10.10.10.103 -d htb.local
```
[Streamio](https://shuciran.github.io/posts/Streamio/#fnref:bloodhound-python)
[[Sizzle#^af63c2]]

### Running Bloodhound
#Note To execute bloodhound we need to run the following commands (one command each line):
```bash
neo4j console 
bloodhound --no-sandbox
```
After extract/get the .json files go to the bloodhound GUI and upload them, then you'll have a bunch of useful information for lateral and horizontal escalation:
![Description](/assets/img/Pasted image 20230125004339.png)
After loading we then can use the bloodhound tool to check for potential pivoting and privilege escalations on the machines. One of the first enumeration techniques that we need to perform is to enumerate what we can do with our current compromised user, for that we need to enter its name on the search bar:
![Description](/assets/img/Pasted image 20230125004842.png)
Then we click on "Mark User as Owned":
![Description](/assets/img/Pasted image 20230125004932.png)
Then we click on user's icon and select any of the options on the search navigation bar specifically the "Analysis" tab and then select the Domain Admin Group:
![Description](/assets/img/Pasted image 20230125005138.png)
If the user is not generated on the graphic that means, there is no direct path to the Domain Admin:
![Description](/assets/img/Pasted image 20230125005421.png)
So we need to keep looking, now with the option "Shortest Path from Owned Principals"
![Description](/assets/img/Pasted image 20230125005540.png)
And now we get something interesting:
![Description](/assets/img/Pasted image 20230125005648.png)
If we right -click on the "AddKeyCredentialLink" and select the Help menu:
![Description](/assets/img/Pasted image 20230125010007.png)
We can see that in order to escalate privileges we need to abuse of Whiskers executable:
![Description](/assets/img/Pasted image 20230125013418.png)
And also it gives the Info to Abuse of such misconfiguration:
![Description](/assets/img/Pasted image 20230125013507.png)
Examples:
[Outdated](https://shuciran.github.io/posts/Outdated/#fnref:bloodhound-execution)

### Clearing the database
All we need to do is click on Menu > Database Info > Clear Database:
![Description](/assets/img/Pasted image 20230129041203.png)