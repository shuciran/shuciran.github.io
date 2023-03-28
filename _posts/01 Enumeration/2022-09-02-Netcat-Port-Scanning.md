---
description: >-
  Enumeration with Netcat.
title: Netcat Port Scanning                   # Add title here
date: 2022-09-02 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Netcat Port Scanning]                     # Change Templates to Writeup
tags: [netcat port scanning]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### TCP Scanning

The -w option specifies the connection timeout in seconds and -z is used to specify zero-I/O mode, which will send no data and is used for scanning:

```bash
nc -nvv -w 1 -z 10.11.1.220 3388-3390
```

### UDP Scanning
Since UDP is stateless and does not involve a three-way handshake, the mechanism behind UDP port scanning is different from TCP.
```bash
nc -nv -u -z -w 1 10.11.1.115 160-162
```
