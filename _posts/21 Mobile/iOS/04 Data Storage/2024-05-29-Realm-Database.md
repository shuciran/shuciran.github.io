---
description: >-
 Realm Database
title:  Realm database          # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Data Storage - Realm DB]                     # Change Templates to Writeup
tags: [mobile, data storage, realm]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Realm DB

Realm DB is an alternative to SQLite for storing structured data in mobile applications. It is object-oriented, which means that the database internally uses objects that map to the mobile application’s classes. Because of this characteristic, using Realm can offer advantages such as simpler code and better performance compared to SQLite.
Data stored in Realm DB can be encrypted, but this option is not enabled by default. To enable encryption, you can specify a 64-byte Realm encryption key in the database configuration, this key must be supplied every time you open the database. Realm DB uses the key to transparently encrypt and decrypt the stored data using an AES cipher. The Realm key must not be stored inside the application binary or in other forms of insecure storage. The iOS Keychain can be leveraged to store and retrieve it securely.
