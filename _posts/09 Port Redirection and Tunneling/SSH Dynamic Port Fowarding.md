### Dynamic Port Forwarding

Now comes the really fun part. SSH _dynamic port forwarding_ allows us to set a local listening port and have it tunnel incoming traffic to any remote destination through the use of a proxy.

In this scenario (similar to the one used in the SSH local port forwarding section), we have compromised a Linux-based target and have elevated our privileges. There do not seem to be any inbound or outbound traffic restrictions on the firewall.

After further enumeration of the compromised Linux client, we discover that in addition to being connected to the current network (10.11.0.x), it has an additional network interface that seems to be connected to a different network (192.168.1.x). On this internal subnet, we have identified a Windows Server 2016 machine that has network shares available.

In the local port forwarding section, we managed to interact with the available shares on the Windows Server 2016 machine; however, that technique was limited to a particular IP address and port. In this example, we would like to target additional ports on the Windows Server 2016 machine, or hosts on the internal network without having to establish different tunnels for each port or host of interest.

We can use ssh -D to specify local dynamic _SOCKS4_ application-level port forwarding (again tunneled within SSH) with the following syntax:

```
ssh -N -D <address to bind to>:<port to bind to> <username>@<SSH server address>
```

With the above syntax in mind, we can create a local SOCKS4 application proxy (-N -D) on our Kali Linux machine on TCP port 8080 (127.0.0.1:8080), which will tunnel all incoming traffic to any host in the target network, through the compromised Linux machine, which we log into as student (student@10.11.0.128):

```
kali@kali:~$ sudo ssh -N -D 127.0.0.1:8080 student@10.11.0.128
student@10.11.0.128's password:
```

Although we have started an application proxy that can route application traffic to the target network through the SSH tunnel, we must somehow direct our reconnaissance and attack tools to use this proxy. We can run any network application through HTTP, SOCKS4, and SOCKS5 proxies with the help of _ProxyChains_.[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/port-redirection-and-tunneling/ssh-tunneling/ssh-dynamic-port-forwarding#fn1)

To configure ProxyChains, we simply edit the main configuration file (/etc/proxychains.conf) and add our SOCKS4 proxy to it:

```
kali@kali:~$ cat /etc/proxychains.conf
...

[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 8080 
```

This configuration is illustrated in Figure 5:

![Figure 5: Dynamic port forwarding diagram](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/d80cf1918150c869299877a80ab7cd45-port_redirection_and_tunneling_diagram_05.png)

To run our tools through our SOCKS4 proxy, we prepend each command with proxychains.

For example, let's attempt to scan the Windows Server 2016 machine on the internal target network using nmap. In this example, we aren't supplying any options to proxychains except for the nmap command and its arguments:

```
kali@kali:~$ sudo proxychains nmap --top-ports=20 -sT -Pn 192.168.1.110
ProxyChains-3.1 (http://proxychains.sf.net)

Starting Nmap 7.60 ( https://nmap.org ) at 2019-04-19 18:18 EEST
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:443-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:23-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:80-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:8080-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:445-<><>-OK
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:135-<><>-OK
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:139-<><>-OK
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:22-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:3389-<><>-OK
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:1723-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:21-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:5900-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:111-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:25-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:53-<><>-OK
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:993-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:3306-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:143-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:995-<--timeout
|S-chain|-<>-127.0.0.1:8080-<><>-192.168.1.110:110-<--timeout
Nmap scan report for 192.168.1.110
Host is up (0.17s latency).

PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   closed ssh
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   open   domain
80/tcp   closed http
110/tcp  closed pop3
111/tcp  closed rpcbind
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
143/tcp  closed imap
443/tcp  closed https
445/tcp  open   microsoft-ds
993/tcp  closed imaps
995/tcp  closed pop3s
1723/tcp closed pptp
3306/tcp closed mysql
3389/tcp open   ms-wbt-server
5900/tcp closed vnc
8080/tcp closed http-proxy

Nmap done: 1 IP address (1 host up) scanned in 3.54 seconds
```

In commands above, ProxyChains worked as expected, routing all of our traffic to the various ports dynamically, without having to supply individual port forwards.

By default, ProxyChains will attempt to read its configuration file first from the current directory, then from the user's $(HOME)/.proxychains directory, and finally from /etc/proxychains.conf. This allows us to run tools through multiple dynamic tunnels, depending on our needs.