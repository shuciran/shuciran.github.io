---
description: >-
 Data Storage Keychain dumper
title:  Data Storage Keychain dumper           # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Data Storage - Keychain Dumper]                     # Change Templates to Writeup
tags: [mobile, data storage, keychain dumper]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### iOS Keychain Dumper

Since the Keychain stores sensitive information, attackers are naturally interested in accessing its contents. We can use the [iOS Keychain Dumper tool](https://github.com/ptoomey3/Keychain-Dumper), to dump the contents of the iOS Keychain on jailbroken devices.

To use the Dumper tool, simply download the keychain_dumper executable and copy it on a jailbroken iOS device. Running it without any options will only dump "Generic" and "Internet" password items. The option "-a" will dump all Keychain items.