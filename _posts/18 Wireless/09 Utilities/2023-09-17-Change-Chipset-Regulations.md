---
description: >-
  Change Chipset Regulations
title:  Change Chipset Regulations           # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Utilities]                     # Change Templates to Writeup
tags: [wireless, chipset regulations, iw]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Change Chipset Regulations

Countries' regulations can be fairly complex, and CRDA sets the radio to operate within the regulations of the operating country. Specifically, it enforces transmit power limits on the radio, prevents the radio from transmitting on restricted frequencies, and abides by any other limitation such as DFS. The `iw reg` command interacts with CRDA to query, and in some cases, change it.

To display the current regulatory domain, we use `iw reg get`:
```bash
kali@kali:~$ sudo iw reg get
global
country MX: DFS-FCC
        (2402 - 2482 @ 40), (N/A, 20), (N/A)
        (5170 - 5250 @ 80), (N/A, 17), (N/A), AUTO-BW
        (5250 - 5330 @ 80), (N/A, 24), (0 ms), DFS, AUTO-BW
        (5490 - 5730 @ 160), (N/A, 24), (0 ms), DFS
        (5735 - 5835 @ 80), (N/A, 30), (N/A)
```
To change or set the regulatory domain, we run `iw reg set <COUNTRY>` where "COUNTRY" is the 2 letter code for the country we are currently in.

