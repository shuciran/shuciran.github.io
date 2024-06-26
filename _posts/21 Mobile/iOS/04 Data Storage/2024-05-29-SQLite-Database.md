---
description: >-
 SQLite Database
title:  SQLite Database           # Add title here
date: 2024-05-29 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Data Storage - SQLite DB]                     # Change Templates to Writeup
tags: [mobile, data storage, sqlite]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### SQLite DB

To save structured data, such as contact information or to-do lists, we can leverage the iOS Core Data framework. It provides a convenient API for storing data in different store types such as an SQLite database. The database file is stored in the app sandbox under `/private/var/mobile/Containers/Data/Application/ {GUID}/Library/Application Support/`

By default, SQLite databases are not encrypted; therefore, they are unsuitable for storing sensitive information. In order to protect the data stored in SQLite databases, we can use SQLCipher, an extension that adds security enhancements for SQLite databases.

SQLCipher works by encrypting the entire SQLite database file using AES. The encryption key is derived from a passphrase, which can be either chosen by the user or automatically generated by the mobile app. Of course, the passphrase itself must be securely stored—for example, hardcoding it into the application would lead to a security vulnerability, as attackers would be able to retrieve it by decompiling the application. Some examples of more secure solutions include:
•	In case the passphrase is chosen by the user, avoid storing it altogether and prompt the user to enter it when the database must be encrypted or decrypted
•	In case the passphrase is automatically generated by the application, leverage the iOS Keychain to save and retrieve it securely
SQLCipher is not natively supported by the Core Data framework but requires using the open-source library Encrypted Core Data. As an alternative, other wrapper libraries around SQLite with SQLCipher support such as FMDB are available.

### SQLiteSpy

While it is possible to use the sqlite3 tool from the commandline on the iOS device, it’s much nicer to copy the database file over to a host system and use visual tools such as SQLiteSpy. 

![SQLiteSpy](/assets/img/Pasted-image-20240529203622.png)

SQLiteSpy is available at [SQLiteSpy](https://www.yunqa.de/delphi/apps/sqlitespy/index).
