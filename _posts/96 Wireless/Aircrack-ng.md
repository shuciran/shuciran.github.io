##### Airodump-ng
Read a pcap and extract the data:
```bash
airodump-ng -r airdrop.pcap
```
Example:
![[Pasted image 20220813235541.png]]

##### Airdecap-ng
Decrypt a pcap file with the password by knowing the ESSID and the password:
```bash
airdecap-ng -e 'main_office_guest' -p 'haishie0ail6hieM' airdrop.pcap
```