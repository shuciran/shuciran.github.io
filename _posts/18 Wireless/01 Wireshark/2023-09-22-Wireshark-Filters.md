---
description: >-
  Wireshark Display Filters
title:  Wireshark Display Filters           # Add title here
date: 2023-09-22 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireshark]                     # Change Templates to Writeup
tags: [wireless, wireshark filters]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireshark Display Filters

```bash
# packets containing certificates (useful in WPA enterprise)
tls.handshake.certificate
# wlan.fc.type have four different values: 0, 1, 2, and 3: Management, Control, Data, and Extension frames, respectively.
wlan.fc.type == 2
# beacon frames
wlan.fc.type_subtype == 0x08
# specify ESSID
wlan.ssid contains "XYZ"
# filter by BSSID
wlan.bssid contains 00:01:20:43:21:12
# X represents frame types: 0 (management), 1 (control), 2 (data), and 3 (extension)
wlan.fc.type == X
# X represents frame subtypes
wlan.fc.subtype == X
# EAPoL frames
wlan.fc.type_subtype in {0x0 0x1 0xb}
# search for a  certain client MAC address
wlan.addr == xx.xx.xx.xx.xx.xx
```

Initial filter
```bash
wlan.bssid contains 00:01:20:43:21:12 and (not subtype beacon) and not (type ctl) and not (subtype probe-req) and not (subtype probe-resp)
```