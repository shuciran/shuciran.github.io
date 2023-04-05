We will use a port forwarding tool called _rinetd_ to redirect traffic on our Kali Linux server. This tool is easy to configure, available in the Kali Linux repositories, and is easily installed with apt:

```
sudo apt update && sudo apt install rinetd
```

The rinetd configuration file, /etc/rinetd.conf, lists forwarding rules that require four parameters, including _bindaddress_ and _bindport_, which define the bound ("listening") IP address and port, and _connectaddress_ and _connectport_, which define the traffic's destination address and port:

```
kali@kali:~$ cat /etc/rinetd.conf
...
# forwarding rules come here
#
# you may specify allow and deny rules after a specific forwarding rule
# to apply to only that forwarding rule
#
# bindadress    bindport  connectaddress  connectport
...
```

For example, we can use rinetd to redirect any traffic received by the Kali web server on port 80 to the google.com IP address we used in our tests. To do this, we will edit the rinetd configuration file and specify the following forwarding rule:

```
kali@kali:~$ cat /etc/rinetd.conf 
...
# bindadress    bindport  connectaddress  connectport
0.0.0.0 80 216.58.207.142 80
...
```

This rule states that all traffic received on port 80 of our Kali Linux server, listening on all interfaces (0.0.0.0), regardless of destination address, will be redirected to 216.58.207.142:80. This is exactly what we want. We can restart the rinetd service with service and confirm that the service is listening on TCP port 80 with ss (socket statistics):

```
kali@kali:~$ sudo service rinetd restart

kali@kali:~$ ss -antp | grep "80"
LISTEN   0   5   0.0.0.0:80     0.0.0.0:*     users:(("rinetd",pid=1886,fd=4))
```

Excellent! The port is listening. For verification, we can connect to port 80 on our Kali Linux virtual machine:

```
student@debian:~$ nc -nvv 10.11.0.4 80
(UNKNOWN) [10.11.0.4] 80 (http) open
GET / HTTP/1.0

HTTP/1.0 200 OK
Date: Mon, 26 Aug 2019 15:46:18 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Server: gws
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Set-Cookie: 1P_JAR=2019-08-26-15; expires=Wed, 25-Sep-2019 15:46:18 GMT; path=/; domain=.google.com
Set-Cookie: NID=188=Hdg-h4aalehFQUxAOvnI87Mtwcq80i07nQqBUfUwDWoXRcqf43KYuCoBEBGmOFmyu0kXyWZCiHj0egWCfCxdote0ScMX6ArouU2jF4DZeeFHBhqZCvLJDV3ysgPzerRkk9pcLi7HEnbeeEn5xR9BgWfz4jvZkjnzYDwlfoL2ivk; expires=Tue, 25-Feb-2020 15:46:18 GMT; path=/; domain=.google.com; HttpOnly
...
```

The connection to our Linux server was successful, and we performed a successful _GET_ request against the web server. As evidenced by the _Set-Cookie_ field, the connection was forwarded properly and we have, in fact, connected to Google's web server.

We can now use this technique to connect from our previously Internet-disconnected Linux client, through the Linux web server, to any Internet-connected host by simply changing the _connectaddress_ and _connectport_ fields in the web server's /etc/rinetd.conf file.

Figure 2 summarizes this process visually:

![Figure 2: Outbound traffic filtering bypass](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/port_redirection_and_tunneling/81e3c06f2dfdaa260fc4eec9c95c2efc-port_redirection_and_tunneling_diagram_02.png)
