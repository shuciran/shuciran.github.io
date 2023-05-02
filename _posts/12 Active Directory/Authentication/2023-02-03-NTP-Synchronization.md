---
description: >-
  Synchronize Attacking Machine with NTP on DC victim machine
title: NTP Synchronization              # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Authentication]          # Change Templates to Writeup
tags: [active directory, authentication, ntp synchronization]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#Note It is possible that sometimes you need to use the host domain (e.g. sizzle.htb)
## NTPDATE
```bash
ntpdate 10.10.11.102
```
## RDATE
```bash
rdate -n 10.10.11.102
```
## DATE
It is also possible to set the date "manually" extracting the date from a web server "Date" header:
```bash
date -s "$(curl -s -k -I https://www.windcorp.htb | grep date | cut -d " " -f2- | tr -d ',')"
```
## Alternative DATE
```bash
# Move the timezone to GMT or the same zone than the AD:
timedatectl set-timezone 'GMT'
# Get and synchronize the hour in real time:
date -s "$(curl -s -k -I https://www.windcorp.htb | grep date | cut -d " " -f2- | tr -d ',')"

Fri Feb  3 10:57:56 AM GMT 2023
# Be sure that it is indeed working:
curl -s -k -I https://www.windcorp.htb | grep date | cut -d " " -f2- | tr -d ','            

Fri 03 Feb 2023 10:57:59 GMT
```
[[Anubis#^bcaafb]]