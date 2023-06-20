Although the Metasploit Framework is preinstalled in Kali Linux, the _postgresql_ service that Metasploit depends on is neither active nor enabled at boot time. We can start the required service with the following command:

```
┌──(kali㉿kali)-[~]
└─$ sudo systemctl start postgresql
```

Next, we can enable the service at boot with systemctl as follows:

```
┌──(kali㉿kali)-[~]
└─$ sudo systemctl enable postgresql
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
```

With the database started, we need to create and initialize the MSF database with msfdb init as shown below.

```
┌──(kali㉿kali)-[~]
└─$ sudo msfdb init
[i] Database already started
[+] Creating database user 'msf'
[+] Creating databases 'msf'
┏━(Message from Kali developers)
┃
┃ We have kept /usr/bin/python pointing to Python 2 for backwards
┃ compatibility. Learn how to change this and avoid this message:
┃ ⇒ https://www.kali.org/docs/general-use/python3-transition/
┃
┗━(Run “touch ~/.hushlogin” to hide this message)
[+] Creating databases 'msf_test'
┏━(Message from Kali developers)
┃
┃ We have kept /usr/bin/python pointing to Python 2 for backwards
┃ compatibility. Learn how to change this and avoid this message:
┃ ⇒ https://www.kali.org/docs/general-use/python3-transition/
┃
┗━(Run “touch ~/.hushlogin” to hide this message)
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

Since Metasploit is under constant development, we should update it as often as possible. Within Kali, we can update Metasploit with apt.

```
┌──(kali㉿kali)-[~]
└─$ sudo apt update; sudo apt install metasploit-framework
```

We can launch the Metasploit command-line interface with msfconsole. The -q option hides the ASCII art banner and Metasploit Framework version output as shown in Listing 5:

```
┌──(kali㉿kali)-[~]
└─$ sudo msfconsole -q
msf6 >
```