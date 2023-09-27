---
description: >-
  Airdecap-ng (Decryption)
title:  Airdecap-ng (Decryption)       # Add title here
date: 2023-09-26 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, airdecap-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Airdecap-ng

Airdecap-ng is useful after we have successfully retrieved the key to a wireless network. We can use it to decrypt WEP, WPA PSK, or WPA2 PSK capture files.

```bash
# Keep the packets targeted to a specific <BSSID> and remove the rest from a cap file (creates a new file)
airdecap-ng -b <MAC> file.pcap

# decrypt saved traffic with a passphrase (check that the passphrase works, we may capture failed logins)
airdecap-ng -b <bssid> -e 'main_office_guest' -p 'haishie0ail6hieM' airdrop.pcap
```