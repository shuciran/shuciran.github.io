## Crackmapexec
```bash
crackmapexec winrm 192.168.197.170 -u users.txt -H hash.txt
```

## Kerbrute
Users file must be only the user not the domain (correct: ksimpson, incorrect: ksimpson@scrm.local)
```bash
kerbrute bruteuser --dc 10.10.11.168 -d scrm.local <users> <password>
```