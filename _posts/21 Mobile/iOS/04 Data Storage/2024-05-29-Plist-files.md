---
description: >-
 Plist file reader
title:  Plist Files           # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Data Storage - Plist files]                     # Change Templates to Writeup
tags: [mobile, data storage, plist, embedded plist]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Plist File

Plist files use an XML format and can be stored in plaintext ASCII or as a proprietary packed file format intended to reduce the file size. These binary files require the use of a supporting editing tool to view the file contents, such as the plutil command line utility on macOS/Linux/iOS or the plist Editor for Windows.

### Plist Editor (Windows)

You can download Windows [Plist Editor](http://www.icopybot.com/plist-editor.htm)

### plutil (Linux)

Installation for Linux:

```bash
apt-get install plutil
```

### Embedded Plist Data

Plist files can store any data and are often used to store embedded plist data within a plist file. This embedded plist data in a plist file can store even more plist data, and so on. Plist viewers including XCode on macOS and Plist Editor for Windows do not interpret embedded plist data, typically displaying binary content in base64 encoded format.

The plistsubtractor tool is used to extract embedded plist data from a plist file, regardless of how deep the plist data is stored. This Python script takes one or more plist files as command line arguments and extracts any embedded plist data in the plist file. The new file is also parsed to determine whether it has any embedded plist data, writing the new data out as a file if it exists.

```bash
$ ls com.apple.nano.plist
com.apple.nano.plist

$ plistsubtractor.py com.apple.nano.plist
Writing com.apple.nano-dndEffectiveOverrides.plist

$ Is -1 com.apple.nano*
-rw-r—r— 1 jwright staff 764 Dec 26 19:35 com.apple.nanodndEffectiveOverrides.plist
-rw-r—r— 1 jwright staff 1046 Dec 26 16:21 com.apple.nano.plist
```

Plistsubtractor is available at [Github-Plistsubstractor](https://github.com/joswrlght/plistsubtractor).

