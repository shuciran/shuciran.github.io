---
description: >-
  Linux Networking Enumeration
title: Networking (Linux)                  # Add title here
date: 2022-10-12 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Linux]                     # Change Templates to Writeup
tags: [linux enumeration, networking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
#### PSPY
[Pspy github download](https://github.com/DominicBreuker/pspy)
Monitoring Services running
```bash
./pspy64
```
Examples:
[Epsilon](https://shuciran.github.io/posts/Epsilon/#fnref:pspy)
#### IP Address
```bash
ifconfig -a
ip address show
ip a s
```

#### DNS
```bash
cat /etc/resolv.conf
```

#### Network connections
```bash
netstat -tulnpa
ss -tulnpwr
lsof -i
watch ss -twurp `connections in live`
```

#### Running services
```bash
ps -aux
ps -ef
```

#### Routing and ARP Tables
```bash
route -n
ip ro show
arp -a 
```

#### Print IPSEC VPN Keys (requires root)
```bash
ip xfrm state list
```

#### Iptables Rules (requires root)
```bash
iptables -L -n
cat /etc/iptables
iptables-save
```

#### Port Scanner
```bash
#!/bin/bash
host=10.5.5.11
for port in {1..65535}; do
    timeout .1 bash -c "echo >/dev/tcp/$host/$port" &&
        echo "port $port is open"
done
echo "Done"
```
