If the postgresql service is running, Metasploit will log findings and information about discovered hosts, services, or credentials in a convenient, accessible database.

In the following listing, the database has been populated with the results of the TCP scan we ran in the previous section. We can display these results with the services command:

```
msf6 auxiliary(scanner/portscan/tcp) > services
Services
========

host            port  proto  name           state  info
----            ----  -----  ----           -----  ----
192.168.120.11  80    tcp                   open   
192.168.120.11  135   tcp                   open   
192.168.120.11  139   tcp                   open   
192.168.120.11  445   tcp                   open   
192.168.120.11  5040  tcp                   open   
192.168.120.11  5357  tcp                   open   
192.168.120.11  7680  tcp                   open   
192.168.120.11  9121  tcp                   open 
```

The basic services command displays all results, but we can also filter by port number (-p), service name (-s), and more as shown in the help output of services -h:

```
msf6 auxiliary(scanner/portscan/tcp) > services -h

Usage: services [-h] [-u] [-a] [-r <proto>] [-p <port1,port2>] [-s <name1,name2>] [-o <filename>] [addr1 addr2 ...]

  -a,--add          Add the services instead of searching
  -d,--delete       Delete the services instead of searching
  -c <col1,col2>    Only show the given columns
  -h,--help         Show this help information
  -s <name>         Name of the service to add
  -p <port>         Search for a list of ports
  -r <protocol>     Protocol type of the service being added [tcp|udp]
  -u,--up           Only show services which are up
  -o <file>         Send output to a file in csv format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -U,--update       Update data for existing service

Available columns: created_at, info, name, port, proto, state, updated_at
```

In addition to a simple TCP port scanner, we can also use the _db_nmap_ wrapper to execute Nmap inside Metasploit and save the findings to the database for ease of access. The db_nmap command has identical syntax to Nmap and is shown below:

```
msf6 auxiliary(scanner/portscan/tcp) > db_nmap
[*] Usage: db_nmap [--save | [--help | -h]] [nmap options]
msf6 auxiliary(scanner/portscan/tcp) > db_nmap 192.168.120.11 -A -Pn
[*] Nmap: 'Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.'
[*] Nmap: Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-21 11:37 EST
[*] Nmap: Nmap scan report for 192.168.120.11
[*] Nmap: Host is up (0.075s latency).
[*] Nmap: Not shown: 995 closed ports
[*] Nmap: PORT     STATE SERVICE       VERSION
[*] Nmap: 80/tcp   open  http
[*] Nmap: | fingerprint-strings:
[*] Nmap: |   FourOhFourRequest:
[*] Nmap: |     HTTP/1.1 404 Not Found
[*] Nmap: |   GenericLines, HTTPOptions, RTSPRequest, SIPOptions:
[*] Nmap: |     HTTP/1.1 400 Bad Request
[*] Nmap: |   GetRequest:
[*] Nmap: |     HTTP/1.1 200 OK
[*] Nmap: |     Content-Type: text/html
[*] Nmap: |     Content-Length: 1228
...
[*] Nmap: |_http-generator: Flexense HTTP v10.0.28
[*] Nmap: |_http-title: Sync Breeze Enterprise @ DESKTOP-6UTT671
[*] Nmap: 135/tcp  open  msrpc         Microsoft Windows RPC
[*] Nmap: 139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
[*] Nmap: 445/tcp  open  microsoft-ds?
[*] Nmap: 5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
...
[*] Nmap: No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
[*] Nmap: TCP/IP fingerprint:
[*] Nmap: OS:SCAN(V=7.91%E=4%D=1/21%OT=80%CT=1%CU=40575%PV=Y%DS=2%DC=T%G=Y%TM=6009AE7
[*] Nmap: OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=105%TI=I%II=I%SS=S%TS=U)OPS
[*] Nmap: OS:(O1=M52DNW8NNS%O2=M52DNW8NNS%O3=M52DNW8%O4=M52DNW8NNS%O5=M52DNW8NNS%O6=M
[*] Nmap: OS:52DNNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%
[*] Nmap: OS:T=80%W=FFFF%O=M52DNW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)
[*] Nmap: OS:T2(R=N)T3(R=N)T4(R=N)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=
[*] Nmap: OS:N)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G
[*] Nmap: OS:)IE(R=Y%DFI=N%T=80%CD=Z)
[*] Nmap: Network Distance: 2 hops
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Host script results:
[*] Nmap: | smb2-security-mode:
[*] Nmap: |   2.02:
[*] Nmap: |_    Message signing enabled but not required
[*] Nmap: | smb2-time:
[*] Nmap: |   date: 2021-01-21T16:40:09
[*] Nmap: |_  start_date: N/A
[*] Nmap: TRACEROUTE (using port 21/tcp)
[*] Nmap: HOP RTT      ADDRESS
[*] Nmap: 1   68.75 ms 192.168.118.1
[*] Nmap: 2   80.66 ms 192.168.120.11
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 159.72 seconds
...
```

To display all discovered hosts up to this point, we can issue the hosts command. As an additional example, we can also list all services running on port 445 with the services -p 445 command.

```
msf6 auxiliary(scanner/portscan/tcp) > hosts

Hosts
=====

address         mac  name             os_name       os_flavor  os_sp  purpose  info  comments
-------         ---  ----             -------       ---------  -----  -------  ----  --------
192.168.120.11       DESKTOP-6UTT671  Windows 2008                    server         
192.168.121.10       CLIENT251        Windows 10    Pro               client         
       

msf6 auxiliary(scanner/portscan/tcp) > services -p 445
Services
========

host            port  proto  name          state  info
----            ----  -----  ----          -----  ----
192.168.120.11  445   tcp    microsoft-ds  open   
```

To help organize content in the database, Metasploit allows us to store information in separate workspaces. When specifying a workspace, we will only see database entries relevant to that workspace, which helps us easily manage data from various enumeration efforts and assignments. We can list the available workspaces with workspace, or provide the name of the workspace as an argument to change to a different workspace as shown in Listing 16.

```
msf6 auxiliary(scanner/portscan/tcp) > workspace
  test
* default
msf6 auxiliary(scanner/portscan/tcp) > workspace test
[*] Workspace: test
msf6 auxiliary(scanner/portscan/tcp) > 
```

To add or delete a workspace, we can use -a or -d respectively, followed by the workspace name.