---
description: >-
  NodeJS Source Code Review
title:  NodeJS Source Code Review             # Add title here
date: 2023-06-19 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, NodeJS Source Code Review]                     # Change Templates to Writeup
tags: [scr, nodejs ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Assessing the Application
```
The existence of bin/www, package.json, and routes/ indicate that this is a NodeJS web application. In particular, package.json identifies a NodeJS project and manages its dependencies.
```

```
The existence of the docker-compose.yml and Dockerfile files indicate that this application is started using Docker containers.
```

### HTTP Routing

Some programming languages and frameworks include routing information directly in the source code. For example, ExpressJS uses this method of routing:
```javascript
var express = require('express');
var router = express.Router();
...

router.get('/login', function(req, res, next) {
  res.render('login', { title: 'Login' });
});
```


### Methods
It is important to know that in a typical NodeJS server side plugin this are the low-hanging-fruit that we need to check first:
-   eval

### Internal 
internal is a reserved word for an "internal" function which then it can be called with "call" reserved word:
Example:

*** Internal ***
Creating the function called "batch"
```bash
internals.batch = function (batchRequest, resultsData, pos, parts, callback) {

    var path = '';
    var error = null;
<SNIP>
```

