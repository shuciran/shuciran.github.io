---
description: >-
  Enumerate System Files on Linux Operating Systems.
title: System (Linux)                 # Add title here
date: 2022-11-10 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration, system]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Distribution
```bash
ls /etc/*-release
cat /etc/os-release
cat /etc/issue
uname -a
cat /proc/version
```

#### Interesting files
```bash
cat /etc/passwd
cat /etc/group
cat /etc/shadow
cat /etc/hosts
ls -lh /var/mail/
ls -lh /usr/bin/
ls -lh /sbin/
```

#### Mounted File Systems
```bash
df -h
mount
cat /etc/fstab
lsblk
```

#### Loaded Kernel Modules
```c
lsmod
/sbin/modinfo <libata> // Change for any library from lsmod command
```

#### Loaded PCI and USB Devices
```bash
lspci
lsusb
```

#### CPU Info
```bash
cat /proc/cpuinfo
cat /proc/meminfo `Memory`
lshw `Hardware Info`
dmesg -T
```
