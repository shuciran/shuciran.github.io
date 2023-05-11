---
description: >-
  Scrambled HTB Machine
title: Scrambled (Medium)                # Add title here
date: 2023-04-24 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Medium]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Scrambled.png                # Add infocard image here for post preview image
---
### Host entries:
```bash
10.10.11.168  DC1.scrm.local   
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- 
- 

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

### Exploitation

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


```bash
kerbrute userenum --dc 10.10.11.168 -d scrm.local /usr/share/seclists/Kerberos/A-ZSurnames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 05/10/23 - Ronnie Flathers @ropnop

2023/05/10 08:42:12 >  Using KDC(s):
2023/05/10 08:42:12 >   10.10.11.168:88

2023/05/10 08:42:12 >  [+] VALID USERNAME:       ASMITH@scrm.local
2023/05/10 08:42:53 >  [+] VALID USERNAME:       JHALL@scrm.local
2023/05/10 08:42:58 >  [+] VALID USERNAME:       KSIMPSON@scrm.local
2023/05/10 08:43:00 >  [+] VALID USERNAME:       KHICKS@scrm.local
2023/05/10 08:43:34 >  [+] VALID USERNAME:       SJENKINS@scrm.local
```

### Root privesc

### Post Exploitation

### Credentials

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### Resources:



