---
description: >-
  Wireless Utilities (Deprecated)
title:  Wireless Utilities (Deprecated)            # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Wireless Utilities (Deprecated)]                     # Change Templates to Writeup
tags: [wireless, iwconfig, utilities, deprecated]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireless Utilities

- `iwlist` allows for the initiation of scanning, listing frequencies, bit rates, and encryption keys.
- `iwspy` provides per-node link quality (not often implemented by drivers).
- `iwpriv` allows for the manipulation of the Wireless Extensions specific to a driver.

#### iwconfig
To see the channel numbers and corresponding frequencies that our wireless interface is able to detect, we can run iwlist with the interface name followed by the frequency parameter:

```bash
kali@kali:~$ sudo iwlist wlan0 frequency
wlan0     14 channels in total; available frequencies :
          Channel 01 : 2.412 GHz
          Channel 02 : 2.417 GHz
          Channel 03 : 2.422 GHz
          Channel 04 : 2.427 GHz
          Channel 05 : 2.432 GHz
          Channel 06 : 2.437 GHz
          Channel 07 : 2.442 GHz
          Channel 08 : 2.447 GHz
          Channel 09 : 2.452 GHz
          Channel 10 : 2.457 GHz
          Channel 11 : 2.462 GHz
          Channel 12 : 2.467 GHz
          Channel 13 : 2.472 GHz
```