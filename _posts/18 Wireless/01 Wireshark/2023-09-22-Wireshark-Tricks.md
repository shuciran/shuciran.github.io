---
description: >-
  Wireshark Tricks
title:  Wireshark Tricks           # Add title here
date: 2023-09-22 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireshark]                     # Change Templates to Writeup
tags: [wireless, wireshark]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Layout

The packet list layout can be rearranged in various ways. Let's select Edit > Preferences > Appearance > Layout to choose another arrangement.

![Wireshark-Layout](/assets/img/Pasted image 20230922001943.png)

### Wireless Toolbar

You can display the toolbar by clicking View > Wireless Toolbar

![Wireless-Toolbar](/assets/img/Pasted image 20230922002547.png)

### Wireshark Hoping

Wireshark doesn't channel hop. To quickly scan all channels on 2.4GHz, we can run the following shell script in the background in a terminal.
```bash
for channel in 1 6 11 2 7 10 3 8 4 9 5
do
  iw dev wlan0mon set channel ${channel}
  sleep 1
done
```