---
description: >-
  Airodump-ng
title:  Airodump-ng         # Add title here
date: 2023-09-26 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, airodump-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Airodump-ng 

Airodump-ng is used to capture raw 802.11 frames.

```bash
# Channel hopping
airodump-ng wlan0

# Specify the channel where airodump listens
airodump-ng --channel 11 --bssid <bssid>

# listen to a single bssid and write output to a file (it creates several files with different formats)
airodump-ng --channel 11 --bssid 10:A7:93:BE:F0:B0 --write <file name>

# scan both 2.4 and 5 GHz simultaneously
airodump-ng wlan0 --band abg

# load capture file in airodump
airodump-ng -r <file.cap>

# show WPS status for WPA networks
airodump-ng wlan0 --wps
```

### Airodump-ng Interactive Mode

The following keys are allowed while executing airodump-ng

[Space] Allows us to freeze the output when we notice something useful on the screen.

[Tab] Enables and disables scrolling through the AP list. 

[🔽,🔼]  When scrolling is enabled we can go up and down with the 🔽 and 🔼 keys.

[M] Cycles through the color options for a selected AP.

[A] Cycles through different displays options.

[S] Cycles through different sorting options

[I] key will invert the sorting 

[D] resets to the default sorting (by power level).
