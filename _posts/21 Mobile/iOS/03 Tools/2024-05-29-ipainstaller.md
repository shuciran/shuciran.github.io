---
description: >-
 ipainstaller functionality
title:  Finding GUID with ipainstaller           # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Tools - ipainstaller]                     # Change Templates to Writeup
tags: [mobile, ipainstaller]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

> Ipainstaller can be obtained by installing the BigBoss Recommended tools through Cydia.
{: .prompt-tip }

### Installing IPAs files

The IPA can be directly installed on the iOS device via the command line with `ipainstaller`. After copying the file to the device, for example via scp, you can execute ipainstaller with the IPA's filename:

```bash
ipainstaller App_name.ipa
```

Usage:
```bash
plutil -xml file.plist
```

### Finding files

An easy way to find local files related to an application on an iOS device is by making use of the ipainstaller tool. We can use this tool to list all the installed apps on the device and determine where the files of these apps are located by making use of the commands displayed below:

```bash
unique-bread:/Applications root# ipainstaller -l
ph.telegra.Telegraph

unique-bread:/Applications root# ipainstaller -i ph.telegra.Telegraph

Identifier: ph.telegra.Telegraph
Version: 21533
Short Version: 7.8.4
Name: Telegram
Display Name: Telegram
Bundle: /private/var/containers/Bundle/Application/5F6902CD-7C32-45D4-BF7C-9BE7FED413D3
Application: /private/var/containers/Bundle/Application/5F6902CD-7C32-45D4-BF7C-9BE7FED413D3/Telegram.app
Data: /private/var/mobile/Containers/Data/Application/A49C6FB6-8D4D-4E64-BBFA-2E56B11DF865
```

![ipainstaller](/assets/img/Pasted-image-20240529190441.png)


