---
description: >-
  Enumerate User's Information on Linux Operating Systems.
title: User Information (Linux)                   # Add title here
date: 2022-08-24 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration, user info]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### Users actions
```bash
who -a `current logged user`
w <user> `info of actual logged users`
whoami
id
last `last logged user`
```

#### All User UID and GID Info
```bash
for user in $(cat /etc/passwd |cut -f1 -d":"); do id $user; done
```

#### All UID 0 Accounts (root)
```bash
cat /etc/passwd |cut -f1,3,4 -d":" |grep "0:0" |cut -f1 -d":" |awk '{print $1}'
```

#### Find Files with “history” In Their Name (.bash_history, etc.)
```bash
find /* -name *.*history* -print 2> /dev/null
```

#### Find Files Owned By A Particular User
```bash 
find / -user <www-data>
``` 

#### Find Files Owned By A Particular Group
```bash
find / -group <sudo>
``` 

#### Find File Types Owned by a Particular User
```bash
find / -user <admin> -name “*.sh”
```
