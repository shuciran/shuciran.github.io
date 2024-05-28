---
description: >-
 Signing and Install IPAs with TrollStore
title:  Signing IPA with TrollStore (Non-jailbroken)           # Add title here
date: 2024-05-27 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Signing IPA - TrollStore]                     # Change Templates to Writeup
tags: [signing, ipa, trollstore]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### TrollStore

For some iOS versions, it is possible to permanently install IPAs even on a non-jailbroken device due to a bug in CoreTrust. The vulnerability is a logical flaw in the processing of the certificate of an IPA, which can trick iOS into installing applications as System apps, after which they remain on the device and can be used indefinitely. It’s even possible to add various entitlements which normally aren’t available to apps, including entitlements reserved for system apps. Unfortunately, there are still a few entitlements that cannot be given to an app installed via TrollStore, namely com.apple.private.es.debugger, dynamic-codesigning, com.apple.private.skip-library-validation, which means it’s not possible to use TrollStore to inject into other apps. This is unfortunate, as this is a requirement for many tweaks, including Frida-server.

* Vulnerability exists in iOS 14.0 - 15.4.1

More information can be found at:
•	[TrollStore](https://github.com/opa334/TrollStore)
•	[CoreTrust](https://worthdoingbadly.com/coretrust/)

![Trollstore](/assets/img/Pasted-image-20240527210527.png)
