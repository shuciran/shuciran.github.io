---
description: >-
  Search Privilige Access Vectors.
title: For Privilege Access (Linux)                  # Add title here
date: 2023-02-18 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux privesc, linux enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Find recursively a string:
```bash
# Find ETSCTF on every file under the current directory
find . -type f -exec grep -H 'ETSCTF' {} \; 2>/dev/null
# Identify any file (not directory) modified in the last day
find / -mtime -l -type f -uid +0 exec ls -al {} \; 2>/dev/null
# Binaries with user SUID
find / -perm -u=s -type f 2>/dev/null
```

##### Capabilities
```bash
getcap -r / 2>/dev/null
```

#### Searching writable directories
```
find / -writable -type d 2>/dev/null
```

##### SETUID
```bash
find / -perm -4000 2>/dev/null
find / -perm -4000 -type f 2>/dev/null `only executables`
find / -perm -u=s -type f 2>/dev/null
```
Examples:
[Epsilon](https://shuciran.github.io/posts/Epsilon/#fnref:setuid)

#### Sudoers
``` 
cat /etc/sudoers
```

#### Find world-writeable files
```bash
find / -perm -0002 -type d 2>/dev/null
```

#### Check current users
```bash
sudo access sudo -l
```

#### Check permissions for files /root directory 
```bash
ls -als /root/*
```

#### Check permissions of dot files/directories
```bash
ls -als /root/.*
```

#### Check for access to users’ .ssh directories
```bash
ls -als /home/*/.ssh
```

#### Check readability of apache/nginx access log
```bash
cat /var/log/apache/access.log
cat /var/log/apache2/access.log
cat /var/log/nginx/access.log
```

#### Search for “user” and “pass” string in Apache Access Log
```bash
cat /var/log/apache/access.log |grep -E “^user|^pass”
```

#### Dump Wireless Pre-Shared
```bash
cat /etc/NetworkManager/system-connections/* | grep -E "^id|^psk"
```

#### Search for "password" string in conf files
```bash
grep "password" /etc/*.conf 2> /dev/null
```

#### PGP Keys
```bash
cat /home/*/.gnupg/secrings.gpgs
```

#### SSH Keys
```bash
cat /home/*/.ssh/id*
```

#### Show any LDAP, Local or NIS Accounts
```bash
getent passwd
```

#### Dump Samba user Database Information
```bash
pdbedit -L -w
pdbedit -L -v
```

#### Kerberos Tickets
```bash
cat /tmp/krb*
```

#### Search for files of .txt extension with name “password”
```bash
find / -name password*.txt 2> /dev/null
```


