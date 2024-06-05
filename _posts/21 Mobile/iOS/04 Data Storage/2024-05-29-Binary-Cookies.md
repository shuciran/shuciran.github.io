---
description: >-
 Binary Cookies
title:  Binary Cookies           # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Data Storage - Binary Cookies]                     # Change Templates to Writeup
tags: [mobile, data storage, binary cookies]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Binary Cookies

Some iOS applications display web content inside WebView components. Webpages often require cookies for mechanisms such as automatic reconnection. iOS applications save these WebView cookies inside binary files called binary cookies.

Due to their binary format, binary cookies are not readable, but can be converted to a readable format.

#### BinaryCookieConverter

BinaryCookieConverter is a Python tool, available from [BinaryCookieConverter](https://github.com/as0ler/BinaryCookieReader), which we can use to read the contents of binary cookies. Run it using the command `python BinaryCookieReader.py` followed by the name of the binary cookie file and it will output the full contents of all the cookies found in the file.

Usage:

```bash
python BinaryCookieReader.py Cookies.binarycookies
```

> Files with suffix .binarycookies are only used on iOS devices
{: .prompt-info }