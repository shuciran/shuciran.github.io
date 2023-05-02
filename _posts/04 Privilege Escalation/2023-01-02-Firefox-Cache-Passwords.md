---
description: >-
  decrypt Mozilla protected passwords
title: Firefox Cache Passwords              # Add title here
date: 2023-01-02 08:00:00 -0600                           # Change the date to match completion date
categories: [04 Privilege Escalation, Firefox Cache Password]                     # Change Templates to Writeup
tags: [firefox, cache password, linux privesc]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Firefox Cache passwords

First go to the path file session for the user:
```powershell
# For Windows:
C:\\Users\\nikk37\\AppData\\Roaming\\Mozilla\\Firefox\\Profiles\\br53rxeg.default-release

# For Linux
/.mozilla/firefox/m6omyf72.default-esr
```
Notice that the name of the folder is randomly generated.

Then extract both files key4.db and logins.json and use the [firepwd.py](https://github.com/lclevy/firepwd) utility:
```python
python3 firepwd.py key4.db logins.json
```
Examples:
[Streamio](https://shuciran.github.io/posts/Streamio/#fnref:firefox-cache-passwords)
[[StreamIO#^9a4b0a]]

> Remember to install the `pycryptodome` for this tool to run correctly `pip install pycryptodome`
{: .prompt-warning }