The Metasploit Framework includes several thousand modules, divided into categories. The categories are displayed on the splash screen summary but we can also view them with the show -h command.

```
msf6 > show -h
[*] Valid parameters for the "show" command are: all, encoders, nops, exploits, payloads, auxiliary, post, plugins, info, options
[*] Additional module-specific parameters are: missing, advanced, evasion, targets, actions
msf6 > 
```

To activate a module, enter use followed by the module name (auxiliary/scanner/portscan/tcp in the example below). At this point, the prompt will indicate the active module. We can use back to move out of the current context and return to the main _msf5_ prompt:

```
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > back
msf6 > 
```

A variation of back is previous, which will switch us back to the previously selected module instead of the main prompt:

```
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > use auxiliary/scanner/portscan/syn
msf6 auxiliary(scanner/portscan/syn) > previous
msf6 auxiliary(scanner/portscan/tcp) > 
```

Most modules require options (show options) before they can be run. We can configure these options with set and unset and can also set and remove _global options_ with setg or unsetg respectively.

```
msf6 auxiliary(scanner/portscan/tcp) > show options

Module options (auxiliary/scanner/portscan/tcp):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   CONCURRENCY  10               yes       The number of concurrent ports to check per host
   DELAY        0                yes       The delay between connections, per thread, in milliseconds
   JITTER       0                yes       The delay jitter factor (maximum value by which to +/- DELAY) in milliseconds.
   PORTS        1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
   RHOSTS                        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   THREADS      1                yes       The number of concurrent threads (max one per host)
   TIMEOUT      1000             yes       The socket connect timeout in milliseconds

msf6 auxiliary(scanner/portscan/tcp) >
```

For example, to perform a scan of our Windows workstation with the scanner/portscan/tcp module, we must first set the remote host IP address (RHOSTS) with the set command.

```
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.120.11
RHOSTS => 192.168.120.11
```

With the module configured, we can run it:

```
msf6 auxiliary(scanner/portscan/tcp) > run

[+] 192.168.120.11:       - 192.168.120.11:80 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:139 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:135 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:445 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:5040 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:5357 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:7680 - TCP OPEN
[+] 192.168.120.11:       - 192.168.120.11:9121 - TCP OPEN
[*] 192.168.120.11:       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```