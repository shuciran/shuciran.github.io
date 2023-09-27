---
description: >-
  Airmon-ng
title:  Airmon-ng         # Add title here
date: 2023-09-25 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Aircrack Suite]                     # Change Templates to Writeup
tags: [wireless, airmon-ng]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Airmon-ng

Airmon-ng is a convenient way to enable and disable monitor mode on various wireless interfaces.

```bash
# Displays the status and information about the wireless interfaces
airmon-ng

# List programs that can interfere with aircrack-ng suite
sudo airmon-ng check 

# Kill processes that can interfere with aircrack-ng suite
sudo airmon-ng check kill

# Create an interface (wlan0mon) in monitor mode from an existing one (wlan0)
sudo airmon-ng start wlan0

# Stop monitor mode
sudo airmon-ng stop wlan0

# Start monitor mode only on channel 2 (only do this when the tool that will be used next doesn't change channels itself)
sudo airmon-ng start wlan0 2

# manually set channel
iw dev wlan0 set channel 13

# Check that we changed the channel correctly
sudo iw dev wlan0 info

# verbose and debug mode
sudo airmon-ng --verbose
sudo airmon-ng --debug
```