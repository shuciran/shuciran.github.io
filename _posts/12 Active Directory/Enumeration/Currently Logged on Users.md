At this point, we can list users along with their group memberships and can easily locate administrative users.

As the next step, we want to find logged-in users that are members of high-value groups since their credentials will be cached in memory and we could steal the credentials and authenticate with them.

If we succeed in compromising one of the _Domain Admins_, we could eventually take over the entire domain (as we will see in a later section). Alternatively, if we can not immediately compromise one of the _Domain Admins_, we must compromise other accounts or machines to eventually gain that level of access.

For example, image below shows that Bob is logged in to CLIENT512 and is a local administrator on all workstations. Alice is logged in to CLIENT621 and is a local administrator on all servers. Finally, Jeff is logged in to SERVER21 and is a member of the Domain Admins group.

![[Pasted image 20221113153134.png]]

If we manage to compromise Bob's account (through a client side attack for example), we could pivot from CLIENT512 to target Alice on CLIENT621. By extension, we may be able to pivot again to compromise Jeff on SERVER21, gaining domain access.

In this type of scenario, we must tailor our enumeration to consider not only _Domain Admins_ but also potential avenues of "chained compromise" including a hunt for a so-called _derivative local admin_

To do this, we need a list of users logged on to a target. We could either interact with the target to detect this directly, or we could track a user's active logon sessions on a domain controller or file server.

The two most reliable Windows functions that can help us to achieve these goals are the _NetWkstaUserEnum_ and _NetSessionEnum_ API. While the former requires administrative permissions and returns the list of all users logged on to a target workstation, the latter can be used from a regular domain user and returns a list of active user sessions on servers such as fileservers or domain controllers.

During an assessment, after compromising a domain machine, we should enumerate every computer in the domain and then use _NetWkstaUserEnum_ against the obtained list of targets. Keep in mind that this API will only list users logged on to a target if we have local administrator privileges on that target.

Alternatively we could focus our efforts on discovering the domain controllers and any potential file servers (based on servers hostnames or open ports) in the network and use _NetSessionEnum_ against these servers in order to enumerate all active users' sessions.

This process would provide us with a good "exploitation map" to follow in order to compromise a domain admin account. However, keep in mind that the results obtained from using these two APIs will vary depending on the current permissions of the logged-in user and the configuration of the domain environment.

As a very basic example, in this section, we will use the _NetWkstaUserEnum_ API to enumerate local users on the Windows 10 client machine and _NetSessionEnum_ to enumerate the users' active sessions on the domain controller.

Calling an operating system API from PowerShell is not completely straightforward. Fortunately, other researchers have presented a technique that simplifies the process and also helps avoid endpoint security detection. The most common solution is the use of [PowerView](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerView/powerview.ps1), a PowerShell script which is a part of the PowerShell Empire framework.

To use it we must first import it:

```
PS C:\Tools\active_directory> Import-Module .\PowerView.ps1
```

> Listing 23 - Installing and importing PowerView

PowerView is quite large but we will only use the _Get-NetLoggedon_ and _Get-NetSession_ functions, which invoke _NetWkstaUserEnum_ and _NetSessionEnum_ respectively.

First, we will enumerate logged-in users with Get-NetLoggedon along with the -ComputerName option to specify the target workstation or server. Since in this case we are targeting the Windows 10 client, we will use -ComputerName client251:

```
PS C:\Tools\active_directory> Get-NetLoggedon -ComputerName client251

wkui1_username wkui1_logon_domain wkui1_oth_domains wkui1_logon_server
-------------- ------------------ ----------------- ------------------
offsec         corp                                 DC01
offsec         corp                                 DC01
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp
CLIENT251$     corp 
```

> Listing 24 - User enumeration using Get-NetLoggedon

The output reveals the expected _offsec_ user account.

Next, let's try to retrieve active sessions on the domain controller DC01. Remember that these sessions are performed against the domain controller when a user logs on, but originate from a specific workstation or server, which is what we are attempting to enumerate.

We can invoke the Get-NetSession function in a similar fashion using the -ComputerName flag. Recall that this function invokes the Win32 API _NetSessionEnum_, which will return all active sessions, in our case from the domain controller.

In Listing 25, the API is invoked against the domain controller _DC01_.

```
PS C:\Tools\active_directory> Get-NetSession -ComputerName dc01


sesi10_cname sesi10_username sesi10_time sesi10_idle_time
------------ --------------- ----------- ----------------
\\192.168.1.111 CLIENT251$                8                8
\\[::1]      DC01$                     6                6
\\192.168.1.111 offsec                    0                0 
```

> Listing 25 - Enumerating active user sessions with Get-NetSession

As expected, the Offsec user has an active session on the domain controller from 192.168.1.111 (the Windows 10 client) due to an active login. The information obtained from the two APIs ended up being the same as we are targeting only a single machine, which also happens to be the one we are executing our script from. In a real Active Directory infrastructure, however, the information gained using each API might differ and would definitely be more helpful.

Now that we can enumerate group membership and determine which machines users are currently logged in to, we have the basic skills needed to begin compromising user accounts with the ultimate goal of gaining domain administrative privileges.