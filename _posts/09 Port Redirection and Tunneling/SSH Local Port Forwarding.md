### Local Port Forwarding

**REMEMBER THAT FOR ALL  THE LOCAL SCENARIOS YOU NEED TO POINT TO YOUR LOCAL ADDRESS AS THE TUNNEL IS REDIRECTING TRAFFIC FROM REMOTE PORT TOWARDS YOURS**

##### Internal Open Port 
There is an internally hosted website on the target VM #1 which is reachable only from the server's local address space:
```bash
ssh -p 2222 -N -L 80:127.0.0.1:80 student@192.168.201.52 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no"
```

##### Remote Port Tunneling to Local Port (SMB)

To pull this off, we will execute an ssh command from our Kali Linux attack machine. We will not technically issue any ssh commands (-N) but will set up port forwarding (with -L), bind port 445 on our local machine (0.0.0.0:445) to port 445 on the Windows Server (192.168.1.110:445) and do this through a session to our original Linux target, logging in as student (student@10.11.0.128):

![Figure 3: Local port forwarding diagram](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/0b19b18b111c4747c0023d6bd79ce85c-port_redirection_and_tunneling_diagram_03.png)

```bash
ssh -N -L [bind_address:]port:host:hostport [username@address]
```

```bash
sudo ssh -N -L 0.0.0.0:445:192.168.1.110:445 student@10.11.0.128
```

Any incoming connection on the Kali Linux box on TCP port 445 will be forwarded to TCP port 445 on the 192.168.1.110 IP address through our compromised Linux client.

We need to make a minor change in our Samba configuration file to set the minimum SMB version to SMBv2 by adding "min protocol = SMB2" to the end of the file as shown in Listing 11. This is because Windows Server 2016 no longer supports SMBv1 by default.

```
kali@kali:~$ sudo nano /etc/samba/smb.conf 

kali@kali:~$ cat /etc/samba/smb.conf 
...
Please note that you also need to set appropriate Unix permissions
# to the drivers directory for these users to have write rights in it
;   write list = root, @lpadmin

min protocol = SMB2

kali@kali:~$ sudo /etc/init.d/smbd restart
[ ok ] Restarting smbd (via systemctl): smbd.service.
```

Finally, we can try to list the remote shares on the Windows Server 2016 machine by pointing the request at our Kali machine.

We will use the _smbclient_ utility, supplying the IP address or NetBIOS name, in this case our local machine (-L 127.0.0.1) and the remote user name (-U Administrator). If everything goes according to plan, after we enter the remote password, all the traffic on that port will be redirected to the Windows machine and we will be presented with the available shares:

```
kali@kali:~# smbclient -L 127.0.0.1 -U Administrator
Unable to initialize messaging context
Enter WORKGROUP\Administrator's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	Data            Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
```



