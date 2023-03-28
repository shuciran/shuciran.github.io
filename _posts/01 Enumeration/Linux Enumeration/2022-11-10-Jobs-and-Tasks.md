---
description: >-
  Jobs and Tasks Enumeration for Linux Operating System.
title: Jobs and Tasks                   # Add title here
date: 2022-11-10 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### List Cron Jobs  
```bash
cat /etc/crontab && ls -al /etc/cron*
```

#### Find World-Writable Cron jobs 
```bash
find /etc/cron* -type f -perm -o+w -exec ls -l {} \;
```

#### Find Cron Jobs Owned by Other Users 
```bash
find /etc/cron* -user <admin>
```
