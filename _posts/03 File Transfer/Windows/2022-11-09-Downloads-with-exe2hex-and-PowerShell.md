---
description: >-
  Downloads with exe2hex and Powershell
title: Downloads with exe2hex and Powershell              # Add title here
date: 2022-11-09 08:00:00 -0600                           # Change the date to match completion date
categories: [03 File Transfer, Downloads with exe2hex and Powershell]                     # Change Templates to Writeup
tags: [file transfer, windows download, exe2hex, powershell]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
This technique is useful to compress the binary we want to transfer, convert it to a hex string, and embed it into a Windows script.

On the Windows machine, we will paste this script into our shell and run it. It will redirect the hex data into powershell.exe, which will assemble it back into a binary. This will be done through a series of non-interactive commands.

```bash
kali@kali:~$ upx -9 nc.exe
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     59392 ->     29696   50.00%    win32/pe     nc.exe
Packed 1 file.

kali@kali:~$ ls -lh nc.exe
-rwxr-xr-x 1 kali kali 29K Sep 18 14:22 nc.exe
```

As we can see, upx has optimized the file size of nc.exe, decreasing it by almost 50%. 

Now that our file is optimized and ready for transfer, we can convert nc.exe to a Windows script (.cmd) to run on the Windows machine, which will convert the file to hex and instruct powershell.exe to assemble it back into binary. We'll use the excellent exe2hex tool for the conversion process:

```bash
kali@kali:~$ exe2hex -x nc.exe -p nc.cmd
[*] exe2hex v1.5.1
[+] Successfully wrote (PoSh) nc.cmd
```

This creates a script named nc.cmd with contents like the following:

```bash
kali@kali:~$ head nc.cmd
echo|set /p="">nc.hex
echo|set /p="4d5a90000300000004000000ffff0000b800000000000000400000000000000000000000000000000000000000000000000000000000000000000000800000000e1fba0e00b409cd21b8014ccd21546869732070726f6772616d2063616e6e6f742062652072756e20696e20444f53206d6f64652e0d0d0a2400000000000000">>nc.hex
echo|set /p="504500004c010300b98eae340000000000000000e0000f010b010500007000000010000000d00000704c010000e000000050010000004000001000000002000004000000000000000400000000000000006001000010000000000000030000000000100000100000000010000010000000000000100000000000000000000000">>nc.hex
...
```

Notice how most of the commands in this script are non-interactive, mostly consisting of echo commands. Towards the end of the script, we find commands that rebuild the nc.exe executable on the target machine:

```bash
...
powershell -Command "$h=Get-Content -readcount 0 -path './nc.hex';$l=$h[0].length;$b=New-Object byte[] ($l/2);$x=0;for ($i=0;$i -le $l-1;$i+=2){$b[$x]=[byte]::Parse($h[0].Substring($i,2),[System.Globalization.NumberStyles]::HexNumber);$x+=1};set-content -encoding byte 'nc.exe' -value $b;Remove-Item -force nc.hex;"
```

When we copy and paste this script into a shell on our Windows machine and run it, we can see that it does, in fact, create a perfectly-working copy of our original nc.exe.

```powershell
...
000000000000000000000000000000000000000000000">>nc.hex

C:\Users\offsec>powershell -Command "$h=Get-Content -readcount 0 -path './nc.hex';$l=$h[0].length;$b=New-Object byte[] ($l/2);$x=0;for ($i=0;$i -le $l-1;$i+=2){$b[$x]=[byte]::Parse($h[0].Substring($i,2),[System.Globalization.NumberStyles]::HexNumber);$x+=1};set-content -encoding byte 'nc.exe' -value $b;Remove-Item -force nc.hex;"
C:\Users\offsec> nc -h
[v1.10 NT]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              detach from console, stealth mode

        -e prog         inbound program to exec [dangerous!!]
...
```