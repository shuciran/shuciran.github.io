---
description: >-
  Hashcat Wireless
title:  Hashcat Wireless      # Add title here
date: 2023-09-27 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Cracking Attacks]                     # Change Templates to Writeup
tags: [wireless, hashcat]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Hashcat

Hashcat is a password cracking tool that was developed to primarily operate on systems with Graphical Processing Units (GPUs) from NVIDIA, AMD, and Intel.

A utility that is specifically relevant for our purposes is cap2hccapx. It exports WPA handshakes from PCAP files to HCCAPx, a format used by hashcat for WPA/WPA2 handshakes.

> To install cap2hccapx run the `sudo apt install hashcat-utils` command. After installation, these utilities are found in `/usr/lib/hashcat-utils`.
{: .prompt-info }

> You can run this command `cp /usr/lib/hashcat-utils/cap2hccapx.bin /usr/bin` to execute the binary from anywhere.
{: .prompt-tip }

> The hashcat module to crack WPA/WPA2 is 2500
{: .prompt-tip }

> We can pause (``p``) and resume (``r``) the hashcat execution
{: .prompt-tip }

```bash
# info about cracking hardware
hashcat -I 

# benchmark of all hash types (very slow)
hashcat -b

# benchmark a single hash type
hashcat -b -m 2500

# extract hashes from a cap file
cap2hccapx.bin file.cap output.hccapx

# Cracking passwords with hashcat
hashcat -m 2500 out.hccapx /usr/share/john/password.lst

# with -d we can choose the cracking device of the listed ones
hashcat64 -m 2500 -d 1 <pcap file> <wordlist>

# with --pot-file we can indicate another path to save the pot file

# install hashcat utilities (found in /usr/lib/hashcat-utils)
sudo apt install hashcat-utils

# convert PCAP file to HCCAPx file with a hashcat util
/usr/lib/hashcat-utils/cap2hccapx.bin wifu-01.cap output.hccapx

```