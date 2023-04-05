So far, we have traversed firewalls based on port filters and stateful inspection. However, certain deep packet content inspection devices may only allow specific protocols. If, for example, the SSH protocol is not allowed, all the tunnels that relied on this protocol would fail.

To demonstrate this, we will consider a new scenario. Similar to our *NIX scenarios, let's assume we have compromised a Linux server through a vulnerability, elevated our privileges to root, and have gained access to the passwords for both the root and student users on the machine.

Even though our compromised Linux server does not actually have deep packet inspection implemented, for the purposes of this section we will assume that a deep packet content inspection feature has been implemented that only allows the HTTP protocol. Unlike the previous scenarios, an SSH-based tunnel will not work here.

In addition, the firewall in this scenario only allows ports 80, 443, and 1234 inbound and outbound. Port 80 and 443 are allowed because this machine is a web server, but 1234 was obviously an oversight since it does not currently map to any listening port in the internal network.

In this case, our goal is to initiate a remote desktop connection from our Kali Linux machine to the Windows Server 2016 through the compromised Linux server using only the HTTP protocol.

We will rely on [_HTTPTunnel_](https://http-tunnel.sourceforge.net) to encapsulate our traffic within HTTP requests, creating an "HTTP tunnel". HTTPTunnel uses a client/server model and we'll need to first install the tool and then run both a client and a server.

The [_stunnel_](https://www.stunnel.org) tool is similar to _HTTPTunnel and can be used in similar ways. It is a multiplatform GNU/GPL-licensed proxy that encrypts arbitrary TCP connections with SSL/TLS.

We can install HTTPtunnel from the Kali Linux repositories as follows:

```
kali@kali:~$ apt-cache search httptunnel
httptunnel - Tunnels a data stream in HTTP requests

kali@kali:~$ sudo apt install httptunnel
...
```

Before diving in, we will describe the traffic flow we are trying to achieve.

First, remember that we have a shell on the internal Linux server. This shell is HTTP-based (which is the only protocol allowed through the firewall) and we are connected to it via TCP port 443 (the vulnerable service port).

We will create a local port forward on this machine bound to port 8888, which will forward all connections to the Windows Server on port 3389, the Remote Desktop port. Note that this port forward is unaffected by the HTTP protocol restriction since both machines are on the same network and the traffic does not traverse the deep packet inspection device. However, the protocol restriction will create a problem for us when we attempt to connect a tunnel from the Linux server to our Internet-based Kali Linux machine. This is where our SSH-based tunnel will be blocked because of the disallowed protocol.

To solve this, we will create an HTTP-based tunnel (a permitted protocol) between the machines using HTTPTunnel. The "input" of this HTTP tunnel will be on our Kali Linux machine (localhost port 8080) and the tunnel will "output" to the compromised Linux machine on listening port 1234 (across the firewall). Here the HTTP requests will be decapsulated, and the traffic will be handed off to the listening port 8888 (still on the compromised Linux server) which, thanks to our SSH-based local forward, is redirected to our Windows target's Remote Desktop port.

When this is set up, we will initiate a Remote Desktop session to our Kali Linux machine's localhost port 8080. The request will be HTTP-encapsulated, sent across the HTTPTunnel as HTTP traffic to port 1234 on the Linux server, decapsulated, and finally sent to our Windows target's remote desktop port.

Take a moment to understand this admittedly complex traffic flow before proceeding. Port forwarding with encapsulation can be complicated because we have to consider firewall rules, protocol limitations, and both inbound and outbound port allocations. It often helps to pause and write a map or flow chart like the one shown below before executing the actual commands. This process is complicated enough without attempting to figure out both logic flow and syntax simultaneously.

![Figure 9: HTTP encapsulation](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/1d20f066892dca8975ae5cf961bd46b8-port_redirection_and_tunneling_diagram_06.png)

To begin building our tunnel, we will create a local SSH-based port forward between our compromised Linux machine and the Windows remote desktop target. Remember, protocol does not matter here (SSH is allowed) as this traffic is unaffected by deep packet inspection on the internal network.

To do this, we will create a local forward (-L) from this machine (127.0.0.1) and will log in as student, using the new password we created post-exploitation. We will forward all requests on port 8888 (0.0.0.0:8888) to the Windows Server's remote desktop port (192.168.1.110:3389):

```
www-data@debian:/$ ssh -L 0.0.0.0:8888:192.168.1.110:3389 student@127.0.0.1
ssh -L 0.0.0.0:8888:192.168.1.110:3389 student@127.0.0.1
Could not create directory '/var/www/.ssh'.
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:RdJnCwlCxEG+c6nShI13N6oykXAbDJkRma3cLtknmJU.
Are you sure you want to continue connecting (yes/no)? yes
yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
student@127.0.0.1's password: lab
...

student@debian:~$ ss -antp | grep "8888"       
ss -antp | grep "8888"
LISTEN     0      128          *:8888                     *:*   
```

Next, we must create an HTTPTunnel out to our Kali Linux machine in order to slip our traffic past the HTTP-only protocol restriction. As mentioned above, HTTPTunnel uses both a client (_htc_) and a server (_hts_).

We will set up the server (hts), which will listen on localhost port 1234, decapsulate the traffic from the incoming HTTP stream, and redirect it to localhost port 8888 (--forward-port localhost:8888) which, thanks to the previous command, is redirected to the Windows target's remote desktop port:

```
student@debian:~$ hts --forward-port localhost:8888 1234
hts --forward-port localhost:8888 1234

student@debian:~$ ps aux | grep hts
ps aux | grep hts
student  12080  0.0  0.0   2420    68 ?        Ss   07:49   0:00 hts --forward-port localhost:8888 1234
student  12084  0.0  0.0   4728   836 pts/4    S+   07:49   0:00 grep hts

student@debian:~$ ss -antp | grep "1234"
ss -antp | grep "1234"
LISTEN  0  1   *:1234   *:*    users:(("hts",pid=12080,fd=4))
```

The ps and ss commands show that the HTTPTunnel server is up and running.

Next, we need an HTTPTunnel client that will take our remote desktop traffic, encapsulate it into an HTTP stream, and send it to the listening HTTPTunnel server. This (htc) command will listen on localhost port 8080 (--forward-port 8080), HTTP-encapsulate the traffic, and forward it across the firewall to our listening HTTPTunnel server on port 1234 (10.11.0.128:1234):

```
kali@kali:~$ htc --forward-port 8080 10.11.0.128:1234

kali@kali:~$ ps aux | grep htc
kali      10051  0.0  0.0   6536    92 ?        Ss   03:33   0:00 htc --forward-port 8080 10.11.0.128:1234
kali      10053  0.0  0.0  12980  1056 pts/0    S+   03:33   0:00 grep htc

kali@kali:~$ ss -antp | grep "8080"
LISTEN  0   0   0.0.0.0:8080    0.0.0.0:*    users:(("htc",pid=2692,fd=4))
```

Again, the ps and ss commands show that the HTTPTunnel client is up and running.

Now, all traffic sent to TCP port 8080 on our Kali Linux machine will be redirected into our HTTPTunnel (where it is HTTP-encapsulated, sent across the firewall to the compromised Linux server and decapsulated) and redirected again to the Windows Server's remote desktop service.

We can validate that this is working by starting Wireshark to sniff the traffic, and verify it is being HTTP-encapsulated, before initiating a remote desktop connection against our Kali Linux machine's listening port 8080:

![Figure 10: RDP login on the Windows Server 2016 machine through the HTTP tunnel](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/f7fdcf772bf6ae8781b19acdbbdd46ad-port_redirection_and_tunneling_06.png)

Excellent! The remote desktop connection was successful.

Inspecting the traffic in Wireshark, we confirm that it is indeed HTTP-encapsulated, and would have bypassed the deep packet content inspection device.

![Figure 11: Inspecting the HTTTP-encapsulated traffic in Wireshark](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/871c2b3b2a9dd257a31a6b745eba2f98-port_redirection_and_tunneling_07.png)