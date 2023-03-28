
### Host entries:
```bash
10.10.11.132
```
If Active Directory => [[Synchronize NTP]] with the domain controller.

# Content

- Web Enumeration
- 

# Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.132
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.132 ()   Status: Up
Host: 10.10.11.132 ()   Ports: 80/open/tcp//http///, 5985/open/tcp//wsman///, 8080/open/tcp//http-proxy///
```
Services and Versions running:
```bash
nmap -p80,5985,8080 -sCV -Pn -n -vvv -oN targeted 10.10.11.132
Nmap scan report for 10.10.11.132
Host is up, received user-set (0.077s latency).
Scanned at 2023-02-06 22:31:51 EST for 13s

PORT     STATE SERVICE REASON  VERSION
80/tcp   open  http    syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
5985/tcp open  http    syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp open  http    syn-ack Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Discovering technologies:
```bash
# Tecnologies for port 80
whatweb http://10.10.11.132                                                  
http://10.10.11.132 [200 OK] Country[RESERVED][ZZ], Email[ideas@object.htb], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.10.11.132], JQuery[2.1.3], Microsoft-IIS[10.0], Modernizr, Script, Title[Mega Engines]

# Tecnologies for port 8080
whatweb http://10.10.11.132:8080
http://10.10.11.132:8080 [403 Forbidden] Cookies[JSESSIONID.487ad83d], Country[RESERVED][ZZ], HTTPServer[Jetty(9.4.43.v20210629)], HttpOnly[JSESSIONID.487ad83d], IP[10.10.11.132], Jenkins[2.317], Jetty[9.4.43.v20210629], Meta-Refresh-Redirect[/login?from=%2F], Script, UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session]
http://10.10.11.132:8080/login?from=%2F [200 OK] Cookies[JSESSIONID.487ad83d], Country[RESERVED][ZZ], HTML5, HTTPServer[Jetty(9.4.43.v20210629)], HttpOnly[JSESSIONID.487ad83d], IP[10.10.11.132], Jenkins[2.317], Jetty[9.4.43.v20210629], PasswordField[j_password], Script[text/javascript], Title[Sign in [Jenkins]], UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session,x-instance-identity], X-Frame-Options[sameorigin]
```


# Exploitation


# Root privesc

# Post Exploitation

# Credentials

# Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

# Resources:



