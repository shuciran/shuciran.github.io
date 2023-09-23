---
description: >-
  rfkill Utility
title:  rfkill Utility           # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Utilities]                     # Change Templates to Writeup
tags: [wireless, rfkill]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### rfkill Utility

rfkill is a tool to enable or disable connected wireless devices.

Let's run rfkill list to display all the enabled Wi-Fi and Bluetooth devices on the system:

```bash
kali@kali:~$ sudo rfkill list
0: hci0: Bluetooth
	Soft blocked: no
	Hard blocked: no
1: phy0: Wireless LAN
	Soft blocked: no
	Hard blocked: no
```

"Soft blocked" refers to a block from rfkill, done in software. "Hard blocked" refers to a physical switch or BIOS parameter for the device. rfkill can only change soft blocks.

A radio can be disabled (soft blocked) using rfkill block followed by the device's ID number that is displayed in the rfkill list command. Using the previous output, we will execute the rfkill command to disable our Wi-Fi device:
```bash
kali@kali:~$ sudo rfkill block 1
kali@kali:~$
```
We run rfkill list 1 to specifically list our disabled Wi-Fi device:
```bash
kali@kali:~$ sudo rfkill list 1
1: phy0: Wireless LAN
	Soft blocked: yes
	Hard blocked: no
```

To re-enable our Wi-Fi device we will run rfkill with the unblock parameter:
```bash
kali@kali:~$ sudo rfkill unblock 1
kali@kali:~$
```

We can disable all radios at the same time with the block all parameter:
```bash
kali@kali:~$ sudo rfkill block all
kali@kali:~$
```

And all the devices can be re-enabled using rfkill with the unblock all parameter.