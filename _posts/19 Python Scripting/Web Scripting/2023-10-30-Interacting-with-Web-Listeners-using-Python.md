---
description: >-
 Interacting with Web Listeners using Python
title:  Interacting with Web Listeners using Python           # Add title here
date: 2023-10-30 08:00:00 -0600                           # Change the date to match completion date
categories: [19 Python Scripting, Web Scripting]                     # Change Templates to Writeup
tags: [Web scripting, web listener, python]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### HTTP Request
The following script will issue an HTTP request:
```python
import requests
from colorama import Fore, Back, Style

proxies = {'http':'http://127.0.0.1:8080','https':'http://127.0.0.1:8080'}

requests.packages.urllib3.\
disable_warnings(requests.packages.urllib3.exceptions.InsecureRequestWarning)
def format_text(title,item):
  cr = '\r\n'
  section_break = cr +  "*" * 20 + cr
  item = str(item)
  text = Style.BRIGHT + Fore.RED + title + Fore.RESET + section_break + item + section_break
  return text

r = requests.get('https://manageengine:8443/',verify=False, proxies=proxies)
print(format_text('r.status_code is: ',r.status_code))
print(format_text('r.headers is: ',r.headers))
print(format_text('r.cookies is: ',r.cookies))
print(format_text('r.text is: ',r.text))
```

