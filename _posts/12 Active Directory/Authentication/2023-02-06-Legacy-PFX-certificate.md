---
description: >-
  Legacy PFX Certificate
title: Legacy PFX Certificate               # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Authentication]          # Change Templates to Writeup
tags: [active directory, authentication, pfx, openssl]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Login via PFX File:
First we need to extract the key and the certificate from the pfx file:
```bash
# Extracting the public certificate:
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out publicCert.pem
# Extracting the private key:
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv-key.pem -nodes
```
Finally you can login with this files using evil-winrm
> (-S) is used if there is only winrm/ssl open port (tcp-5986):
{: .prompt-warning }
```bash
evil-winrm -i 10.10.11.152 -c publicCert.pem -k priv-key.pem -S
```
Examples:
[Timelapse](https://shuciran.github.io/posts/Timelapse/#fnref:pfx-certificate)

References:
[How to Extract Certificate and Private Key from PFX](https://tecadmin.net/extract-private-key-and-certificate-files-from-pfx-file/) 

[IBM PFX Explanation](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file#r_extratsslcert__keypwd)