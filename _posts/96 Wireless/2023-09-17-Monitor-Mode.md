---
description: >-
  Wireless Monitor Mode Interface
title:  Wireless Monitor Mode Interface            # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Wireless Monitor Mode Interface ]                     # Change Templates to Writeup
tags: [wireless, iw, monitor mode]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireless Monitor Mode

First, we add an interface with the interface option and the add parameter followed by its name (wlan0mon). Lastly the type option with monitor places our new interface in monitor mode:
```bash
kali@kali:~$ sudo iw dev wlan0 interface add wlan0mon type monitor
```

With the new interface created, we need to bring it up with ip (newly created interfaces are down by default):
```bash
sudo ip link set wlan0mon up
```
Using the iw dev info command, we will be able to inspect our newly created monitor mode interface:
```bash
kali@kali:~$ sudo iw dev wlan0mon info
Interface wlan0mon
	ifindex 4
	wdev 0x1
	addr 0c:0c:ac:ab:a9:08
	type monitor
	wiphy 0
	channel 11 (2462 MHz), width: 20 MHz, center1: 2462 MHz
```
Lastly we can verify our card is in monitor mode by starting a sniffer, `tcpdump`, to capture wireless frames:
```bash
kali@kali:~$ sudo tcpdump -i wlan0mon
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlan0mon, link-type IEEE802_11_RADIO (802.11 plus radiotap header), capture size 262144 bytes
13:39:17.873700 2964927396us tsft 1.0 Mb/s 2412 MHz 11b -20dB signal antenna 1 [bit 14] Beacon (wifu) [1.0* 2.0* 5.5* 11.0* 9.0 18.0 36.0 54.0 Mbit] ESS CH: 3, PRIVACY[|802.11]
```
> Once we have finished with our VAP, we will want to delete it with the iw command and the del option. Once we've done that, let's confirm it worked with info:
{: .prompt-warning }

```bash
kali@kali:~$ sudo iw dev wlan0mon interface del
kali@kali:~$ sudo iw dev wlan0mon info
command failed: No such device (-19)
```