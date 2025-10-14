---
description: >-
   Windows Registry
title: Windows Registry        # Add title here
date: 2025-09-01 08:00:00 -0600                           # Change the date to match completion date
categories: [22 Evasion Techniques, Windows Registry]          # Change Templates to Writeup
tags: [evasion, windows, registry]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png          # Add infocard image here for post preview image
---

### Windows Registry

Many programming languages support the concept of local and global variables, where local variables are limited in scope and global variables are usable anywhere in the code. An operating system needs global variables in much the same manner. Windows uses the registry to store many of these.

The registry contains important information that can be abused during attacks, and some modifications may allow us to bypass specific defenses.

The registry is effectively a database that consists of a massive number of keys with associated values. These keys are sorted hierarchically using subkeys.

At the root, multiple registry hives contain logical divisions of registry keys. Information related to the current user is stored in the HKEY_CURRENT_USER (HKCU) hive, while information related to the operating system itself is stored in the HKEY_LOCAL_MACHINE (HKLM) hive.


> The HKEY_CURRENT_USER hive is writable by the current user while modification of the HKEY_LOCAL_MACHINE hive requires administrative privileges.
{: .prompt-tip }

We can interface with the registry both programmatically through the Win32 APIs as well as through the GUI with tools like the Registry Editor (regedit).

Since a 64-bit version of Windows can execute 32-bit applications each registry hive contains a duplicate section called Wow6432Node which stores the appropriate 32-bit settings.

The registry is used extensively by the operating system and a variety of applications. As penetration testers, we can obtain various reconnaissance information from it or modify it to improve attacks or perform evasion.
