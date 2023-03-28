---
description: >-
  Network Enumeration tools
title: Network Enumeration                   # Add title here
date: 2022-08-16 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Network Enumeration]                     # Change Templates to Writeup
tags: [network enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### ARP-SCAN
This tool sends a ARP requests to a given IP or network and retrieves the MAC address:

```bash
arp-scan -I tap0 -g 10.142.111.0/24
```

### FPING
Reconnaisance of alive hosts:
```bash
fping -I ens33 -g 10.10.0.0/24 -a 2>/dev/null
```

### NMAP
Reconnaisance of alive hosts:
```bash
nmap -sn 10.10.0.0/24
```
