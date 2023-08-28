---
description: >-
  PHP Source Code Review
title:  PHP Source Code Review             # Add title here
date: 2023-06-22 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, PHP Source Code Review]                     # Change Templates to Writeup
tags: [scr, php ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

We decided to enumerate all pages we could access without authentication using a grep search and used the results as a starting point for our analysis.

```bash
grep -rnw /var/www/html/ATutor -e "^.*user_location.*public.*" --color
```

If at the beginning of the php script the `public` word means that this resource can be reacher without authentication.
```php
$_user_location = 'public';
```

> Any time we see variable names such as query or qry, or function names that contain the string search, our first instinct should be to follow the path and see where the code takes us. It may lead us to nothing or it may lead to code that properly handles user-controlled data, leaving us nothing to work with. Nevertheless, even in a worst case scenario, we could learn how the application handles user input, which can save us time later on when we encounter similar situations.
{: .prompt-tip}


> If the name of a variable starts with `$` it is not a global variable being initialized, instead it might be a function saved on a variable, this sometimes leads us to find a pontential vulnerability due to developer's bad practices.
{: .prompt-tip}

```text
An important item to note here is that the called function name is stored in a variable called $addslashes and that we are not calling the native PHP addslashes function
```

As it turns out, we can use inline comments in MySQL as a valid space! For example, the following SQL query is, in fact, completely valid in MySQL.
```bash
mysql> select/**/1;
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set (0.01 sec)
```

> Session tokens are always an interesting item to keep track of as they are used in unexpected ways at times. We'll make a note of that.
{: .prompt-tip}