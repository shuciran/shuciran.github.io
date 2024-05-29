---
description: >-
 Setup iOS for Mobile Assessment
title:  Setup for iOS Mobile Assessment           # Add title here
date: 2024-05-27 08:00:00 -0600                           # Change the date to match completion date
categories: [21 Mobile, Setting up iOS]                     # Change Templates to Writeup
tags: [iOS setup]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Initial Requirements

Apple device with iOS version 14.6 or prior. It is highly recommended to fabric restore the device or at least backup the information if any.
Clean USB to flash checkn1x.

#WARNING:  If you are having any issue while using this guide, go to the troubleshooting section which can assist on some scenarios that you might face.

#### Jailbreak

This is a guide to configure an Apple device in such a way that you have full control of it, also indicates some useful tools and techniques to perform a successful Mobile Application Security Assessment.

There are two types of jailbreak Untethered and Tethered this guide only shows the last one.

#WARNING:  A Tethered jailbreak is a temporary jailbreak and requires the device to be connected to a computer every time the device needs a restart. The jailbreak is reversed otherwise. Consider always having  your phone charged while using this method.

#### Checkn1x

This guide shows the installation guide of Checkn1x, a Linux-based distribution for jailbreaking iOS devices with checkra1n you can download the iso from here: 
Reference: https://github.com/asineth0/checkn1x/releases

#### Etcher
This is a tool approved by Checkn1x itself to flash the iso file on an USB, you can download the executable from here:

Reference: https://www.balena.io/etcher

Once installed, proceed to flash the ISO file on your USB, it is very user friendly all you need to do is choose the .iso location, select the USB and flash it:



If by any chance you receive the following error, try to reboot your machine:



#### Checkra1n
Connect your USB and boot the checn1x from your phyiscal PC, which will prepare the environment that will install Checkra1n on the Apple device.

First choose the second option by pressing Alt + F2 to run Checkn1x on interactive mode:


Move to the "Options" tab and click enter:



Then choose the options as shown in the image below, this allow you to install Checkra1n on higher versions of iOS like 14.3:


Afterwards, connect the device and you'll receive a message to trust the device on your iOS, click on allow and you'll see the following screen:



Click on "Start" and then you'll receive instructions about how to configure the device for the jailbreaken, please notice that the instructions might vary depending on the device, be sure to read them prior to follow them as it is a bit tricky:



If the final message throws a success message you can disconnect the device, otherwise you can try to repeat the previous steps, if by any chance you are not capable, you can go to the "Troubleshooting" section of this guide, also you can select the "safe mode" option if the jailbreak is not being applied correctly.

Upon Jailbreaken an iOS device, the unofficial app store "Cydia" can be installed, which provides user access to download other unapproved apps:



In order to check if the jailbreaken was successfull you can try to access via SSH to the device, from Cydia application go to the "Search" (1) tab, search for "OpenSSH"(2) and install it (3).


Lastly, access the cellphone from your kalynx via SSH; credentials are root:alpine. If you are able to do so, then the device is successfully jailbroken.

ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" root@192.168.1.92



#### Burp Suite Certificate Installation

First step is to configure your iPhone proxy by going to Settings > Wi-Fi > choose your Wi-Fi network > Proxy configuration > and then choose "Manual" option and then enter the IP where Burpsuite is listening and its port:

 Be sure that your Burp Suite is listening on all interfaces:


Download your Burpsuite Certificate as usual from http://burp and then go to Settings > General > Profile, choose the PortSwigger CA Configuration Profile and then install it:



Finally you need to go to Settings > General > About > Certificate Trust Settings and enable Port Swigger CA option:



#### Frida

From Cydia application you can install Frida repository (not installed by default), click on "Sources" (1), then click on "Edit" (2):



Then click on "Add" (1) and type the frida url (2):

Reference: https://build.frida.re



Finally search the application on the main menu of Cydia and install Frida.

Note: If you are running an iPhone 6+ you can install Frida for 32-bit devices.
           If you are running an iPhone 11+ you can install Frida for A12+ devices.
           

#### Objection
Objection enables us to assess an iOS app in an environment using Frida and it makes short work of re-signing the IPA, installing the app, and other tasks. 

From Kalynx, run the following command:

sudo pip3 install objection

Resources:
https://www.secjuice.com/objection-frida-guide/

Darwin CC tools

Useful set of tools like otool and lipo, to install just search "Darwin CC tools" on Cydia and install it:

#### MobSF

For static analysis with MobSF you need either a linux or a Mac, Kalynx supports perfectly the installation, this are the requirements for it:

sudo apt install python3-dev python3-venv python3-pip build-essential libffi-dev libssl-dev libxml2-dev libxslt1-dev libjpeg62-turbo-dev zlib1g-dev wkhtmltopdf

git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git

Then go to the MobSF folder, execute the "setup.py" script to install the dependencies required, finally start the service with "run.py" script and connect to it via web browser on http://localhost:8000

#### 3uTools

For apps that cannot be put on the App Store, they can be signed with a certificate or signed with an Apple ID and installed on the device normally, you can follow this step-by-step guide:

Note> Only works if device have been jailbreaked

Reference: http://www.3u.com/tutorial/articles/10212/how-to-use-ipa-signature#c

Once that your app is signed, you can install it on your iOS device, go to "iDevice" (1), click on "Apps" (2) and "Import & Install ipa":


#### SSLKillSwitch
An SSL Pinning Bypass technique, in order to install it you need to follow this instructions:

	• Open Cydia and install wget.

	• Ssh in to your jailbroken device and download the package in /tmp folder from the following link using wget command:
Reference: https://github.com/nabla-c0d3/ssl-kill-switch2/releases/tag/0.14

iPhone:/tmp root# wget https://github.com/nabla-c0d3/ssl-kill-switch2/releases/download/0.14/com.nablac0d3.sslkillswitch2_0.14.deb

	• Install the file with dpkg:
iPhone:/tmp root# dpkg -i com.nablac0d3.sslkillswitch2_0.14.deb

	• If you get an error then fire the following command and try again:
iPhone:/tmp root# apt --fix-broken install

	• Now respring your iDevice:
iPhone:~ root# killall -9 backboardd

	• Finally open iOS settings app, scroll down and locate SSL Kill Switch 2. Open it and enable "Disable Certificate Validation"

WARNING: This tool might not work in some cases.

#### Troubleshooting

1. Etcher
A) USB with Checkn1x corrupted
Sometimes while using the checkn1x there are some errors that could avoid the jailbreak, in such cases is better to format the USB and install checkn1x again, however the USB is not formattable as a normal USB, to do so use the following guide to help you format the USB to its original state:

	• Windos + R > diskpart
	• list disk
	• select disk n (here “n” is the disk number of USB drive listed on the previous command)
	• clean
	• create partition primary
	• format fs=ntfs quick

B) Etcher not flashing the USB
If Etcher throws the following error, a simple solution is to restart the PC:

2. Checkn1x

A) Cydia installation unsuccessful
Sometimes the Checkra1n is not correctly installed and during Cydia installation the application could throw the following messages:

This means that the Checkra1n installation was unsuccessful so you will need to restore your apple device, you can do it directly from checkra1n by clicking on "Restore System" but this might cause an outage on your device known as "Black Screen of Death" so it's highly recommended to do it directly from the settings on the iPhone itself.

B) Black Screen of Death (DFU Mode)
Sometimes if you restore your iOS device with the checkra1n feature or if the jailbreak was unsuccessful, the screen might turns out black and while connecting your device to the checkra1n it might shows an error as in the image below. The solution is a hard reset on the device, you can found the way to do it on this video (tested on iphone 8 and iphone 6): 
Reference: iPhone 12: Black Screen or Blank Screen? Screen Won't Turn On? 2 Fixes

WARNING:  Keep in mind that this solution will erase your data.

3. Frida
A) Frida Error

If while using command frida-ps and trying to reach the device the error on the screenshot below appears, follow the steps on Frida installation section in this same guide:


B) Frida and Objection not working
If kali is not reaching the frida and shows the following error:

It might be that the port used by default is not reachable, in such cases you can start frida server from SSH.

And then run the following command:

iPhone:~ root# frida-server -l 0.0.0.0:9999


Finally you'll be able to run both Frida and Objection by pointing to this port:
