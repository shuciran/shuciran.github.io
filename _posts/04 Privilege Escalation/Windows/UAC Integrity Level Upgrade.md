Even while logged in as an administrative user, the account will have two security tokens, one running at a medium integrity level and the other at high integrity level. UAC acts as the separation mechanism between those two integrity levels.

To see integrity levels in action, let's first login as the admin user, open a command prompt, and run the whoami /groups command:

```
c:\Users\admin>whoami /groups

GROUP INFORMATION
-----------------

Group Name                              Type             SID          Attributes
======================================================== ============ ================
Everyone                                Well-known group S-1-1-0      Mandatory group,
NT AUTHORITY\Local account and member   Well-known group S-1-5-114    Group used for d
BUILTIN\Administrators                  Alias            S-1-5-32-544 Group used for d
BUILTIN\Users                           Alias            S-1-5-32-545 Mandatory group,
NT AUTHORITY\INTERACTIVE                Well-known group S-1-5-4      Mandatory group,
CONSOLE LOGON                           Well-known group S-1-2-1      Mandatory group,
NT AUTHORITY\Authenticated Users        Well-known group S-1-5-11     Mandatory group,
NT AUTHORITY\This Organization          Well-known group S-1-5-15     Mandatory group,
NT AUTHORITY\Local account              Well-known group S-1-5-113    Mandatory group,
LOCAL                                   Well-known group S-1-2-0      Mandatory group,
NT AUTHORITY\NTLM Authentication        Well-known group S-1-5-64-10  Mandatory group,
Mandatory Label\Medium Mandatory Level  Label            S-1-16-8192
```

As reported on the last line of output, this command prompt is currently operating at a Medium integrity level.

Let's attempt to change the password for the admin user from this command prompt:

```
C:\Users\admin> net user admin Ev!lpass
System error 5 has occurred.

Access is denied.
```

The request is denied, even though we are logged in as an administrative user.

In order to change the admin user's password, we must switch to a high integrity level even if we are logged in with an administrative user. In our example, one way to do this is through powershell.exe with the _Start-Process_ cmdlet specifying the "Run as administrator" option:

```
C:\Users\admin>powershell.exe Start-Process cmd.exe -Verb runAs
```

After submitting this command and accepting the UAC prompt, we are presented with a new high integrity cmd.exe process.

Let's check our integrity level using the whoami utility using the /groups argument and attempt to change the password again:

```
C:\Windows\system32> whoami /groups
GROUP INFORMATION
-----------------

Group Name                              Type             SID          Attributes
======================================================== ============ ================
Everyone                                Well-known group S-1-1-0      Mandatory group,
NT AUTHORITY\Local account and member   Well-known group S-1-5-114    Mandatory group,
BUILTIN\Administrators                  Alias            S-1-5-32-544 Mandatory group,
BUILTIN\Users                           Alias            S-1-5-32-545 Mandatory group,
NT AUTHORITY\INTERACTIVE                Well-known group S-1-5-4      Mandatory group,
CONSOLE LOGON                           Well-known group S-1-2-1      Mandatory group,
NT AUTHORITY\Authenticated Users        Well-known group S-1-5-11     Mandatory group,
NT AUTHORITY\This Organization          Well-known group S-1-5-15     Mandatory group,
NT AUTHORITY\Local account              Well-known group S-1-5-113    Mandatory group,
LOCAL                                   Well-known group S-1-2-0      Mandatory group,
NT AUTHORITY\NTLM Authentication        Well-known group S-1-5-64-10  Mandatory group,
Mandatory Label\High Mandatory Level    Label            S-1-16-12288

C:\Windows\system32> net user admin Ev!lpass
The command completed successfully.
```

This time, we are running at a high integrity level and the password change is successful.