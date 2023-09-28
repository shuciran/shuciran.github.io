---
description: >-
  Airolib-ng (Cracking PMKs)
title:  Airolib-ng (Cracking PMKs)         # Add title here
date: 2023-09-26 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, airolib-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Airolib-ng

Airolib-ng is a tool designed to store and manage ESSID and password lists, compute their Pairwise Master Keys (PMKs) and use them in WPA/WPA2 cracking through sqlite3.

```bash
# create a text file containing the ESSID of the target AP
echo wifu > essid.txt

# import the text file into an airolib-ng database
airolib-ng wifu.sqlite --import essid essid.txt

# info about database (ESSIDs and stored passwords)
airolib-ng wifu.sqlite --stats

# import a dictionary of passwords (ignores those shorter than 8 chars and larger than 63 chars, since they are not valid WPA passphrases)
airolib-ng wifu.sqlite --import passwd /usr/share/john/password.lst

# calculate the PMK corresponding to each inported password
airolib-ng wifu.sqlite --batch

#crack using a db
aircrack-ng -r wifu.sqlite wpa1-01.cap
```