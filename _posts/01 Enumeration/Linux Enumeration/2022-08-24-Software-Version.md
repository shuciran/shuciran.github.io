---
description: >-
  Enumerate Software on Linux Operating Systems.
title: Software Version                  # Add title here
date: 2022-08-24 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Get MySQL Version
```bash
mysql –version
```

#### Get sudo Version
```bash
sudo -V
```

#### Get Apache2 Version 
```bash
apache2 -v 
```

#### Get CouchDB Version 
```bash
couchdb -V 
```

#### Get Postgres Version 
```bash
psql -V
```

#### List All Packages Installed and Versions 
```bash
dpkg -l
```
