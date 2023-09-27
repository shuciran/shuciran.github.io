---
description: >-
  Aircrack-ng (Cracking)
title:  Aircrack-ng (Cracking)       # Add title here
date: 2023-09-26 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, aircrack-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Aircrack-ng 

Aircrack-ng is considered an offline attack since it works with packet captures and doesn't require interaction with any Wi-Fi device. It can crack WEP and WPA/WPA2 networks that use pre-shared keys or PMKID.

```bash
# Estimates how many passphrases our CPU can crack approximately
aircrack-ng -S  

# DON'T use a dictionary for WEP files!!!!
aircrack-ng wep.cap

# crack a handshake saved in a cap file:
aircrack-ng -w <path to wordlist> -e <ESSID> -b <ap bssid> file.pcap
aircrack-ng -w /usr/share/john/password.lst -e <ESSID> -b <ap bssid> file.cap

# crack using a db created with airolib (precomputed PMKs)
aircrack-ng -r wifu.sqlite wpa1-01.cap
```

> If in a capture file an AP has hidden name but we find it in another way, we need to pass both arguments to aircrack-ng, -b and -e, so that it can match a BSSID to an ESSID
{: .prompt-warning }