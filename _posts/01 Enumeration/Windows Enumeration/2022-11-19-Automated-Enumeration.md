---
description: >-
  Automated Enumeration on Windows Operating Systems.
title: Automated Enumeration                   # Add title here
date: 2022-11-19 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Windows]                     # Change Templates to Writeup
tags: [windows enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### WinPeas
On Windows, one automated script is [WinPeas](https://github.com/carlospolop/PEASS-ng/releases/tag/20221113)

### Windows-Privesc-Check
Another automated script is _windows-privesc-check_, which can be found in the [windows-privesc-check](https://github.com/pentestmonkey/windows-privesc-check) 

We'll specify the self-explanatory --dump to view output, and -G to list groups.

```
windows-privesc-check2.exe --dump -G
```

