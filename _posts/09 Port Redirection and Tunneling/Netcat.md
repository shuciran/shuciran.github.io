### Transfer File
Output the exit of the file towards the netcat listener on the victim machine:
```bash
nc -nv 10.11.0.22 4444 < /usr/share/windows-resources/binaries/wget.exe
```
Then redirect the traffic towards the destination file:
```bash
nc -nlvp 4444 > incoming.exe
```