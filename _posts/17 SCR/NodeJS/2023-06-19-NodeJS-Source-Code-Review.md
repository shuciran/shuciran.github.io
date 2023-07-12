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