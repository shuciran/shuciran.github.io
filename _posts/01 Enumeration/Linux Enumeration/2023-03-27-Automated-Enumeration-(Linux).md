---
description: >-
  Automated Enumeration for Linux Operating System. 
title: Automated Enumeration (Linux)                 # Add title here
date: 2023-02-18 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration, automatization]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### LinPeas
We can use [linpeas](https://github.com/carlospolop/PEASS-ng/releases/tag/20221106) on UNIX derivatives such as Linux. 
```bash
./linpeas.sh | tee output.txt
```

### Unix_privesc_check
We can use [_unix_privesc_check_](https://pentestmonkey.net/tools/audit/unix-privesc-check) on UNIX derivatives such as Linux. 
The script supports "standard" and "detailed" mode.
```
./unix-privesc-check standard > output.txt
```

### PSPY
Download from [PSPY](https://github.com/DominicBreuker/pspy)
