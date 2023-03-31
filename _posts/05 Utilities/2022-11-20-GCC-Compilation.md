---
description: >-
  GCC Compilation
title: GCC Compilation              # Add title here
date: 2022-11-20 08:00:00 -0600                           # Change the date to match completion date
categories: [05 Utilities, GCC Compilation]                     # Change Templates to Writeup
tags: [utilities, gcc compilation]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Basic Compilation
```bash
gcc -o exploit exploit.c
```

### 32-Bit Compilation
```bash
gcc -m32 -Wl,--hash-style=both -o exploit exploit.c
```