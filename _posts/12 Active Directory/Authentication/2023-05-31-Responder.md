---
description: >-
  Responder
title: Responder               # Add title here
date: 2023-02-03 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Enumeration]          # Change Templates to Writeup
tags: [active directory, enumeration, responder, llmnr]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### LLMNR (Link-Local Multicast Name Resolution)

What is LLMNR?
- Link-Local Multicast Name Resolution.
- Used to identify hosts when DNS fails to do so.
- Previously known as NBT-NS.
- The main drawback is that the services use a system username and an NTLMv2 hash to which they respond.

### Responder
> [!] Responder must be run as root.
{: .prompt-danger }
Tool to poisoning the network and catch authentication requests such as NTLM, NTLMv2, etc.
```bash
responder -I tun0
```
> Remember that if you want to capture correctly an NTLM hash, you need to have port tcp-445 available for Responder to work correctly.
{: .prompt-tip }
[Intelligence](https://shuciran.github.io/posts/Intelligence/#fnref:responder)
[[Anubis#^2015a8]]
