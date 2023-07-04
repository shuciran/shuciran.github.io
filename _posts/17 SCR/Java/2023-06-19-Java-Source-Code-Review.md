---
description: >-
  Java Source Code Review
title:  Java Source Code Review             # Add title here
date: 2023-06-19 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, Java Source Code Review]                     # Change Templates to Writeup
tags: [scr, java ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Methods
It is important to know that in a typical Java servlet, we can easily identify the HTTP request handler functions that handle each HTTP request type due to their constant and unique names.

These methods are named as follows:
-   _doGet_
-   _doPost_
-   _doPut_
-   _doDelete_
-   _doCopy_
-   _doOptions_

### Java Recognize
A quick Google search leads us to a file extensions explanation page, which states that the .do extension is typically a URL mapping scheme for compiled Java code.