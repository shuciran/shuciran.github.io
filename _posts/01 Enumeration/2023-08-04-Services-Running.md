---
description: >-
  Services Running on both windows and linux
title: Services Running                 # Add title here
date: 2022-08-04 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Services Running]                     # Change Templates to Writeup
tags: [services running]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

For Windows and Linux the following command shows TCP services running:
```bash
netstat -ano -p tcp
```

Also for linux you can use the following command:
```bash
ss -tulnp
```

Examples:
[[StreamIO#^1d2840]]
[[Antique#^dac47f]]

While within a machine you can enumerate the ports open locally with this script:
```bash
#!/bin/bash

for port in $(seq 1 65535); do
        timeout 1 bash -c "echo '' > /dev/tcp/127.0.0.1/$port" &>/dev/null && echo "[+] Puerto activo $port" &

done
```
Examples:
[[Flustered]]
