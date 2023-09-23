---
description: >-
  Installing Wireless Card Drivers
title:  Installing Wireless Cards Drivers           # Add title here
date: 2023-09-22 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Preparing The Environment]                     # Change Templates to Writeup
tags: [wireless, drivers]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Installing Wireless Card Drivers
To install drivers for a specific wireless card we need to upgrade the and update our linux machine as follows:

```bash
sudo apt update && sudo apt upgrade
sudo reboot
sudo apt install -y linux-headers-$(uname -r) build-essential bc dkms git libelf-dev rfkill iw
mkdir -p ~/src
cd ~/src
git clone https://github.com/morrownr/8814au.git
cd ~/src/8814au
sudo ./install-driver.sh
sudo reboot
```

> The 3rd command won't work until the first reboot.
{: .prompt-warning }