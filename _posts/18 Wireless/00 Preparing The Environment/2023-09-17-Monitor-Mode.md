---
description: >-
  Wireless Monitor Mode Interface
title:  Wireless Monitor Mode Interface            # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Preparing The Environment]                     # Change Templates to Writeup
tags: [wireless, iw, monitor mode]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireless Monitor Mode Interface

First, we add an interface with the interface option and the add parameter followed by its name (wlan0mon). Lastly the type option with monitor places our new interface in monitor mode:
```bash
kali@kali:~$ sudo iw dev wlan0 interface add wlan0mon type monitor
```

With the new interface created, we need to bring it up with ip (newly created interfaces are down by default):
```bash
sudo ip link set wlan0mon up
```
Using the iw dev info command, we will be able to inspect our newly created monitor mode interface:
```bash
kali@kali:~$ sudo iw dev wlan0mon info
Interface wlan0mon
        ifindex 8
        wdev 0x2
        addr 00:0f:00:69:34:61
        type monitor
        wiphy 0
        txpower 20.00 dBm
```
Lastly we can verify our card is in monitor mode by starting a sniffer, `tcpdump`, to capture wireless frames:
```bash
kali@kali:~$ sudo tcpdump -i wlan0mon
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlan0mon, link-type IEEE802_11_RADIO (802.11 plus radiotap header), capture size 262144 bytes
12:52:20.753528 1.0 Mb/s 2432 MHz 11b -41dBm signal antenna 1 Beacon (Not_Of_Your_Buzzinez) [1.0* 2.0* 5.5* 11.0* 18.0 24.0 36.0 54.0 Mbit] ESS CH: 5, PRIVACY
```
> Once we have finished with our VAP, we will want to delete it with the iw command and the del option. Once we've done that, let's confirm it worked with info:
{: .prompt-warning }

```bash
kali@kali:~$ sudo iw dev wlan0mon interface del
kali@kali:~$ sudo iw dev wlan0mon info
command failed: No such device (-19)
```

### Change between monitor and manage mode

#### Monitor mode
##### airmon-ng
```bash
airmon-ng start wlan0
```
##### Manually
``` bash
# Use the following command to set interface in monitor mode.
iw dev <interface> set monitor none

# If this gives you device busy error, then do the following:
ifconfig <interface> down
iw dev <interface> set monitor none
ifconfig <interface> up

# Also you can try with the deprecated commands:
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up
```

#### Managed mode
> Needed for connecting to networks!!!
{: .prompt-warning }

##### airmon-ng
``sudo airmon-ng stop wlan0mon``

##### Manually
```bash
ifconfig mon0 down
ifconfig mon0 mode managed
ifconfig mon0 up
```

We can disconnect and reconnect the adapter. With `iwconfig` we can see the mode of the interface.
To connect to a network we need to reestart NetworkManager, if we killed it previously with `airmon-ng check kill`:
```bash 
sudo service NetworkManager start
```

If the network uses mac filtering we cannot connect. It can be blacklist or whitelist. If it's blacklist we can use any non blacklisted MAC. If it's whitelisted we need to use the MAC of a connected client.
A symptom of MAC filtering is that the network is OPEN or we have a password and still can't connect

> Sometimes changed MACs don't stay when trying to connect to the network.
{: .prompt-warning }

