---
description: >-
  Airodump-ng
title:  Airodump-ng         # Add title here
date: 2023-09-25 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, airodump-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Airodump-ng 

Airodump-ng is meant to be a full-blown tool to capture wireless

```bash
# Channel hopping
airodump-ng wlan0

# Specify the channel where airodump listens
airodump-ng --channel 11 --bssid <bssid>

# listen to a single bssid and write output to a file (it creates several files with different formats)
airodump-ng --channel 11 --bssid <bssid> --write <file name>

# scan both 2.4 and 5 GHz simultaneously
airodump-ng wlan0 --band abg

# load capture file in airodump
airodump-ng -r <file.cap>

# show WPS status for WPA networks
airodump-ng wlan0 --wps
```