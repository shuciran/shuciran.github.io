---
description: >-
  Loading and Unloading Wireless Drivers
title:  Loading and Unloading Wireless Drivers             # Add title here
date: 2023-09-17 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Utilities]                     # Change Templates to Writeup
tags: [wireless, drivers, unloading and loading ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Loading and Unloading Wireless Drivers

This guide explains the process to load and unload a wireless driver. If there are two or more devices using the same driver, can cause unexpected results while using them for Wireless Pentesting, therefor we are able to unload (blacklist) one of them.

Determine the driver with `airmon-ng`
```bash
kali@kali:~$ sudo airmon-ng
PHY     Interface       Driver          Chipset
phy0    wlan0           ath9k_htc       Qualcomm Atheros Communications AR9271 802.11n
```

Listing system USB devices and shows detailed information for each device:

```bash
kali@kali:~# sudo lsusb -vv

Bus 001 Device 002: ID 0cf3:9271 Qualcomm Atheros Communications AR9271 802.11n
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          255 Vendor Specific Class
  bDeviceSubClass       255 Vendor Specific Subclass
  bDeviceProtocol       255 Vendor Specific Protocol
  bMaxPacketSize0        64
  idVendor           0x0cf3 Qualcomm Atheros Communications
  idProduct          0x9271 AR9271 802.11n
  bcdDevice            1.08
  iManufacturer          16 ATHEROS
  iProduct               32 USB2.0 WLAN
  iSerial                48 12345
  bNumConfigurations      1
...
```
Determining dependencies, compatibility, and firmware requirements are displayed with the `modinfo` command and the name of the driver. Running `modinfo` for the driver displays the following output:
```bash
kali@kali:~$ sudo modinfo ath9k_htc
filename:       /lib/modules/4.16.0-kali2-amd64/kernel/drivers/net/wireless/ath/ath9k/ath9k_htc.ko
firmware:       ath9k_htc/htc_9271-1.4.0.fw
firmware:       ath9k_htc/htc_7010-1.4.0.fw
description:    Atheros driver 802.11n HTC based wireless devices
license:        Dual BSD/GPL
author:         Atheros Communications
alias:          usb:v0CF3p20FFd*dc*dsc*dp*ic*isc*ip*in*
...
alias:          usb:v0CF3p1006d*dc*dsc*dp*ic*isc*ip*in*
alias:          usb:v0CF3p9271d*dc*dsc*dp*ic*isc*ip*in*
depends:        mac80211,ath9k_hw,ath9k_common,ath,cfg80211,usbcore
retpoline:      Y
intree:         Y
name:           ath9k_htc
vermagic:       4.16.0-kali2-amd64 SMP mod_unload modversions
parm:           debug:Debugging mask (uint)
...
parm:           blink:Enable LED blink on activity (int)
```
`lsmod` lists all the loaded modules as well as the dependencies of each module. Running the command with the driver loaded outputs the following:
```bash
kali@kali:~$ lsmod
Module                  Size  Used by
ath9k_htc              81920  0
ath9k_common           20480  1 ath9k_htc
ath9k_hw              487424  2 ath9k_htc,ath9k_common
ath                    32768  3 ath9k_htc,ath9k_hw,ath9k_common
mac80211              802816  1 ath9k_htc
cfg80211              737280  4 ath9k_htc,mac80211,ath,ath9k_common
rfkill                 28672  3 cfg80211
uhci_hcd               49152  0
ehci_pci               16384  0
ehci_hcd               94208  1 ehci_pci
ata_piix               36864  0
mptscsih               36864  1 mptspi
usbcore               290816  5 ath9k_htc,usbhid,ehci_hcd,uhci_hcd,ehci_pci
usb_common             16384  1 usbcore
...
```

A good example of when to use blacklisting would be the case where an open source driver and the closed source vendor drivers are both present on the system. If we run `modinfo` on both of them, we will see they share similar IDs. There should only be one driver claiming a device at a time, so we have to blacklist one of them. If we don't, the two drivers will fight for the same resource, causing unexpected results.

`lsmod` lists all the loaded modules as well as the dependencies of each module. Running the command with the ath9k_htc driver loaded outputs the following:

```bash
kali@kali:~$ lsmod
Module                  Size  Used by
ath9k_htc              81920  0
ath9k_common           20480  1 ath9k_htc
ath9k_hw              487424  2 ath9k_htc,ath9k_common
ath                    32768  3 ath9k_htc,ath9k_hw,ath9k_common
mac80211              802816  1 ath9k_htc
cfg80211              737280  4 ath9k_htc,mac80211,ath,ath9k_common
rfkill                 28672  3 cfg80211
uhci_hcd               49152  0
ehci_pci               16384  0
ehci_hcd               94208  1 ehci_pci
ata_piix               36864  0
mptscsih               36864  1 mptspi
usbcore               290816  5 ath9k_htc,usbhid,ehci_hcd,uhci_hcd,ehci_pci
usb_common             16384  1 usbcore
...
```

With our `lsmod` output, we can start removing modules that are not needed by other drivers. If we are unsure which module to remove next, we can run lsmod again and find one that isn't used by any other.

```bash
kali@kali:~$ sudo rmmod ath9k_htc ath9k_common ath9k_hw ath
```