---
description: >-
  User Enumeration Gathering
title:  User Enumeration Gathering             # Add title here
date: 2022-09-03 08:00:00 -0600                           # Change the date to match completion date
categories: [08 Passive Reconnaissance, User Enumeration Gathering]                     # Change Templates to Writeup
tags: [passive reconnaissance, osint, user enum gathering]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
## Email Harvesting
### The Harvester

```bash
theHarvester -d megacorpone.com -b google
```
> The Harvester doesn't works really well on newer versions of Kali, use the docker image if that is the case.
{: .prompt-tip }
```bash
docker run -ti --rm theharvester:latest
```

### HaveIbeenPwned
Tool to get information about e-mails that are involved on 
[have i been pwned](https://haveibeenpwned.com)

### Snusbase
Along with HaveIbeenPwned this site is useful to collect passwords (one week costs $7 USD life time account $333).
> If you find an e-mail with HaveIBeenPwned and you are in a Red Team, this tools might be really helpful to gather leaked passwords.
{: .prompt-tip }

[Snusbase](https://snusbase.com/login)

### CrossLinked
[crosslinked](https://github.com/m8sec/CrossLinked.git) is a tool to gather emails from LinkedIn using both first name and last name 

> You should first find a valid e-mail to craft users such as `shuciran.naka@mycompany.com` 
{: .prompt-tip }

Then the format of the command would be like:
```python
python3 crosslinked.py -f '{first}.{last}@mycompany.com' mycompany
```