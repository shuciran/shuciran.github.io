---
description: >-
  Wireless Utilities
title:  Wireless Utilities            # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Wireless Utilities ]                     # Change Templates to Writeup
tags: [wireless, iw, iwconfig, utilities]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireless Utilities

Even though we could still use iwconfig and other tools are deprecated and we shouldn't use them anymore. The iw utility and its variety of options is the only command we need for configuring a Wi-Fi device.

### iw 

`iw list` will provide us with lots of detailed information about the wireless devices and their capabilities such as if the wireless adapter supports monitor mode:
```bash
kali@kali:~$ sudo iw list
Wiphy phy0
	...
	Supported interface modes:
		 * IBSS
		 * managed
		 * AP
		 * AP/VLAN
		 * monitor
		 * mesh point
		 * P2P-client
		 * P2P-GO
		 * outside context of a BSS
```
To get a listing of wireless access points that are within range of our wireless card, we will use `iw` with the `dev wlan0` option
```bash
kali@kali:~$ sudo iw dev wlan0 scan | grep SSID
	SSID: wifu
	SSID: 6F36E6
```

Listing wireless access points that are within range of our wireless card:
```bash
kali@kali:~$ sudo iw dev wlan0 scan | grep SSID
	SSID: wifu
	SSID: 6F36E6
```
Listing wireless channel numbers that are within range of our wireless card:
```bash
kali@kali:~$ sudo iw dev wlan0 scan | egrep "DS Parameter set|SSID:"
	SSID: wifu
	DS Parameter set: channel 3
	SSID: 6F36E6
	DS Parameter set: channel 11
```
To display the current regulatory domain, we use iw reg get:
```bash
kali@kali:~$ sudo iw reg get
global
country 00: DFS-UNSET
	(2402 - 2472 @ 40), (6, 20), (N/A)
	(2457 - 2482 @ 20), (6, 20), (N/A), AUTO-BW, PASSIVE-SCAN
	(2474 - 2494 @ 20), (6, 20), (N/A), NO-OFDM, PASSIVE-SCAN
	(5170 - 5250 @ 80), (6, 20), (N/A), AUTO-BW, PASSIVE-SCAN
	(5250 - 5330 @ 80), (6, 20), (0 ms), DFS, AUTO-BW, PASSIVE-SCAN
	(5490 - 5730 @ 160), (6, 20), (0 ms), DFS, PASSIVE-SCAN
	(5735 - 5835 @ 80), (6, 20), (N/A), PASSIVE-SCAN
	(57240 - 63720 @ 2160), (N/A, 0), (N/A)
```
By default, Kali is set to global regulatory domain (00).

#### iwconfig and other utilities (deprecated)
- `iwlist` allows for the initiation of scanning, listing frequencies, bit rates, and encryption keys.
- `iwspy` provides per-node link quality (not often implemented by drivers).
- `iwpriv` allows for the manipulation of the Wireless Extensions specific to a driver.

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