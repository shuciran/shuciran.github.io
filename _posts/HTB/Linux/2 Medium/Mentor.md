
### Host entries:
```bash
10.10.11.193    mentorquotes.htb api.mentorquotes.htb
```

# Content

- Web Enumeration
- Subdomain Enumeration
- API Documentation analysis
- SNMP Enumeration

# Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.193
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.193 ()   Status: Up
Host: 10.10.11.193 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -Pn -n -vvv -oN targeted 10.10.11.193
Nmap scan report for 10.10.11.193
Host is up, received user-set (0.076s latency).
Scanned at 2023-02-04 20:54:34 GMT for 8s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c73bfc3cf9ceee8b4818d5d1af8ec2bb (ECDSA)
| ecdsa-sha2-nistp256 
...
|   256 4440084c0ecbd4f18e7eeda85c68a4f7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJjv9f3Jbxj42smHEXcChFPMNh1bqlAFHLi4Nr7w9fdv
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://mentorquotes.htb/
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Initial reconnaissance for UDP ports
```bash
extractUDPPorts allUDPPorts

[*] Extracting information...

        [*] IP Address: 10.10.11.193
        [*] Open ports: 161
```
# Exploitation


# Root privesc

# Post Exploitation

# Credentials
```bash
# Possible credentials:
/usr/local/bin/login.py kj23sadkj123as0-d213

```
# Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

# Resources:



