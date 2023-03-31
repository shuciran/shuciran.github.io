---
description: >-
  Subversion service enumeration.
title: Subversion (tcp-3690)                 # Add title here
date: 2023-03-27 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Subversion (tcp-3690)]                     # Change Templates to Writeup
tags: [subversion]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Enumeration commands

```bash
svn ls svn://10.10.10.203 #list
svn log svn://10.10.10.203 #Commit history
svn checkout svn://10.10.10.203 #Download the repository
svn up -r 2 #Go to revision 2 inside the checkout folder
```

Example:
[Worker](https://shuciran.github.io/posts/Worker/#fnref:subversion)