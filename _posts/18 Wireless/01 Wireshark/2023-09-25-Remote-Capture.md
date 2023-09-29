---
description: >-
  Remote Capture
title:  Remote Capture           # Add title here
date: 2023-09-25 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireshark]                     # Change Templates to Writeup
tags: [wireless, wireshark, remote capture]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### TCPDUMP 
(-i) interface 
(-w) output file (- for STDOUT)
(-U) output each packet as it arrives
```bash
sudo tcpdump -i wlan0mon -w - -U
```

### DUMPCAP
(-P) output data on pcap format
```bash
sudo dumpcap -w - -P -i wlan0mon
```

### TSHARK
```bash
sudo tshark -w - -i wlan0mon
```

#### TSHARK EAP [^tshark-eap]
```bash
# show packets in a file
sudo tshark -r wpa-eap-tls.pcap 

# show captured packets applying a filter for packets containing certificates exchanged during handshaek
sudo tshark -r wpa-eap-tls.pcap -Y "tls.handshake.certificate" 

# show all data (-x)
sudo tshark -r wpa-eap-tls.pcap -Y "tls.handshake.certificate" -x

# show all fields in capture files (the ones filtered with -Y)
tshark -r b64.pcap -Y "tls.handshake.certificate" -T pdml

# show a specific field (in this case, the certificate)
tshark -r b64.pcap -Y "tls.handshake.certificate" -T fields -e "tls.handshake.certificate" 

# full plaintext dump of packet (the same that you can see on wireshark)
tshark -nr b64.pcap -2 -R "ssl.handshake.certificate" -V

# in JSON format, easier to read:
tshark -nr b64.pcap -2 -R "ssl.handshake.certificate" -T json -V
```

### Piping packets to wireshark
```bash
sudo tcpdump -U -w - -i wlan0mon | wireshark -k -i -
```

### SSH Remotely command
```bash
ssh root@10.11.0.196 "sudo -S tcpdump -U -w - -i wlan0mon" | sudo wireshark -k -i -
```

[^tshark-eap]: tshark eap filters