---
description: >-
  Squid Proxy Enumeration
title: Squid Proxy (tcp-3128)                 # Add title here
date: 2023-01-23 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Squid Proxy (tcp-3128)]                     # Change Templates to Writeup
tags: [squid proxy enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
If a Squid Proxy is enabled we can configure our proxychains to enumerate services behind port tcp-3128, all we need to do is configure the /etc/proxychains.conf file as follows:

```bash
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
#socks4         127.0.0.1 1080
http 10.10.10.224 3128
```

Then to enumerate the services we use the command "proxychains", keep in mind that nmap using proxychains only works for -sT or three hand shake enumeration:
```bash
proxychains -q nmap -sT -Pn -n -vvv 127.0.0.1
```
Example:
[Tentacle](https://shuciran.github.io/posts/Tentacle/#fnref:squid-proxy)
