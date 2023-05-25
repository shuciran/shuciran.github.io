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
### PSPY
[Pspy github download](https://github.com/DominicBreuker/pspy)
Monitoring Services running
```bash
./pspy64
```
Examples:
[Epsilon](https://shuciran.github.io/posts/Epsilon/#fnref:pspy)
### IP Address
```bash
ifconfig -a
ip address show
ip a s
```

### DNS
```bash
cat /etc/resolv.conf
```

### Network connections
```bash
netstat -tulnpa
ss -tulnpwr
lsof -i
watch ss -twurp `connections in live`
```

### Running services
```bash
ps -aux
ps -ef
```

### Routing and ARP Tables
```bash
route -n
ip ro show
arp -a 
```

### Print IPSEC VPN Keys (requires root)
```bash
ip xfrm state list
```

### Iptables Rules (requires root)
```bash
iptables -L -n
cat /etc/iptables
iptables-save
```
> Sometimes a one-liner is slowly, to play with threads we can create a script and disown the process of this one-liner in such way that the loop does not run one instruction at a time, this can be achieved with amperson (&)
{: .prompt-tip }

### Host Scanner
```bash
#!/bin/bash
for i in $(seq 1 255):
do
        timeout 1 bash -c "ping -c 1 192.168.122.$i &>/dev/null" && echo "[+] IP 192.168.122.$i active" & done; wait
```
Examples:
[Fulcrum](https://shuciran.github.io/posts/Fulcrum/#fnref:hosts-scanner)

### Port Scanner
```bash
#!/bin/bash
host=192.168.122.228
for port in {1..65535}; do
    timeout .1 bash -c "echo >/dev/tcp/$host/$port" && echo "port $port is open" &
done
echo "Done"
```
Examples:
[Fulcrum](https://shuciran.github.io/posts/Fulcrum/#fnref:port-scanner)

### Subnet Port Scanner

> This is a scanner using proxychains, if you don't have a proxychains configuration, remove the `proxychains` command.
{: .prompt-warning }

```bash
#!/bin/bash

for port in 21 22 23 25 80 88 443 445 8080 8081 9001; do
        for i in $(seq 1 254); do
                proxychains -q timeout 1 bash -c "echo '' > /dev/tcp/10.241.251.$i/$port" 2>/dev/null && echo "[+] Port $port - OPEN in Host: 10.241.251.$i" &
        done;
done;
```
Examples:
[Tentacle](https://shuciran.github.io/posts/Tentacle/#fnref:subnet-scanner)
