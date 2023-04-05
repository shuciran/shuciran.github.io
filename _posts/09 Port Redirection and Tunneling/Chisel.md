[Chisel](https://github.com/jpillora/chisel/releases)

### Remote port forwarding
Chisel as client:
```powershell
# Single Port
.\\chisel.exe client 10.10.16.4:1337 R:1433:localhost:1433
# All the ports
./chisel client 10.10.14.3:1234 R:127.0.0.1:socks
```
Examples:
[[StreamIO#^b193f4]]
[[Anubis#^e12b6c]]

Chisel as server:
```bash
./chisel_1.7.7_linux_amd64 server --port 1337 --reverse
```
Examples:
[[StreamIO#^c5cdca]]

### Forward SOCKS Proxy
Chisel as client:
```bash
./chisel.sh server -p 2345 --socks5
```
Chisel as server:
```bash
./chisel_1.7.7_linux_amd64 client 10.10.11.107:2345 <PROXY_PORT>:socks
```
Additionally /etc/proxychains.conf file needs to be setted as follows:
```bash
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 <PROXY_PORT>
```
Finally if this is an HTTP service we need to configure the foxyproxy with the port choosen:
![[Pasted image 20220711213232.png]]
Examples:
[[Antique#^45b307]]