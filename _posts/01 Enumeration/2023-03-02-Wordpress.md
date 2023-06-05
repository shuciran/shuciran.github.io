---
description: >-
  Wordpress Enumeration
title: Wordpress                 # Add title here
date: 2023-03-02 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Wordpress]                     # Change Templates to Writeup
tags: [Wordpress enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Default paths:
-   `index.php`
-   `license.txt` contains useful information such as the version WordPress installed.
-   `wp-activate.php` is used for the email activation process when setting up a new WordPress site.
-   Login folders (may be renamed to hide it):
    - `/wp-admin/login.php`
    - `/wp-admin/wp-login.php`
    - `/login.php`
    - `/wp-login.php`
 
- `xmlrpc.php` is a file that represents a feature of WordPress that enables data to be transmitted with HTTP acting as the transport mechanism and XML as the encoding mechanism. This type of communication has been replaced by the WordPress [REST API](https://developer.wordpress.org/rest-api/reference).

- The `wp-content` folder is the main directory where plugins and themes are stored.
  
- `wp-content/plugins/` directory with the installed plugins

- `wp-content/uploads/` Is the directory where any files uploaded to the platform are stored.

- `wp-includes/` This is the directory where core files are stored, such as certificates, fonts, JavaScript files, and widgets.

Examples:
[[Backdoor#^34b6a9]]
[[Backdoor#^11157a]]

### WPScan
For a thorough scan, we will need to provide the URL of the target (--url) and configure the enumerate option (--enumerate) to include "All Plugins" (ap), "All Themes" (at), "Config backups" (cb), and "Db exports" (dbe). 
```bash
wpscan --url sandbox.local --enumerate ap,at,cb,dbe 
```
#### Aggresive Mode
```bash
wpscan --url http://example.com/ --plugins-detection aggressive
```

### Brute Force
```bash
wpscan --url http://locker.ptd/ -U MB6zE2vkSCV -P /usr/share/wordlists/rockyou.txt
```

> You can also add your API Token from wpscan directly by creating a user account to do so, you need to pass the `--api-token` parameter.
{: .prompt-tip }
