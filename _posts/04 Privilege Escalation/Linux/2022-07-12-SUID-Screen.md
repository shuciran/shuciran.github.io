---
description: >-
  SUID Screen Exploitation
title: SUID Screen Exploitation             # Add title here
date: 2022-07-12 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Linux - SUID Screen Exploitation]                     # Change Templates to Writeup
tags: [linux privesc, suid, screen command]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Screen
If screen is running as SUID you can look for a dettached session and use it to escalate privileges, first run the following command:
```bash
ps -aux | grep screen
```
If there is indeed a screen command being executed, then in order to get the session as the user executing run the following command:
```bash
screen -r <name of the session>/
```
Examples:
[Backdoor](https://shuciran.github.io/posts/Backdoor/#fnref:suid-screen)