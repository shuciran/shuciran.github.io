---
description: >-
  Enumerate and abuse Linux Directory Structure.
title: Linux Abusing Directory Structure                  # Add title here
date: 2022-08-24 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
##### /proc/(PID)/cmdline
This file shows the parameters passed to the kernel at the time it is started. It looks like the following:

```python
---------------------------------------------------
[*] PATH: /proc/816/cmdline
[*] Total lenght: 181
b'/proc/816/cmdline/proc/816/cmdline/proc/816/cmdline/bin/sh\\x00-c\\x00while true;
do su user -c "cd /home/user;gdbserver --once 0.0.0.0:1337 /bin/true;";
done\\x00<script>window.close()</script>'
---------------------------------------------------
```
Examples:
[[Backdoor#^4e1616]]

##### /proc/net/fib_trie
This path saves the whole structure of the interfaces, is useful to grab the current IP and also to check if whether or not we are interacting with a docker, and it looks as follows:
```bash
Main: 
	+-- 0.0.0.0/1 2 0 2 
		+-- 0.0.0.0/4 2 0 2 
			|-- 0.0.0.0 /0 universe UNICAST 
			+-- 10.10.10.0/23 2 0 1 
				|-- 10.10.10.0 
					/32 link BROADCAST 
					/23 link UNICAST 
				|-- 10.10.11.125 
					/32 host LOCAL 
				|-- 10.10.11.255 
					/32 link BROADCAST 
		+-- 127.0.0.0/8 2 0 2 
			+-- 127.0.0.0/31 1 0 0 
				|-- 127.0.0.0 
					/32 link BROADCAST 
					/8 host LOCAL 
				|-- 127.0.0.1 /32 host LOCAL 
				|-- 127.255.255.255 
					/32 link BROADCAST 
Local: 
	+-- 0.0.0.0/1 2 0 2 
		+-- 0.0.0.0/4 2 0 2 
			|-- 0.0.0.0 
				/0 universe UNICAST 
			+-- 10.10.10.0/23 2 0 1 
				|-- 10.10.10.0 
					/32 link BROADCAST 
					/23 link UNICAST 
				|-- 10.10.11.125 
					/32 host LOCAL 
				|-- 10.10.11.255 
					/32 link BROADCAST 
		+-- 127.0.0.0/8 2 0 2 
			+-- 127.0.0.0/31 1 0 0 
				|-- 127.0.0.0 
					/32 link BROADCAST 
					/8 host LOCAL 
				|-- 127.0.0.1 
					/32 host LOCAL 
			|-- 127.255.255.255 
				/32 link BROADCAST
```

##### Mail Directory
```bash
/var/mail/
```

##### Installed applications

```bash
/usr/bin/
/usr/sbin
```
