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

### Java Reconnaissance
A quick Google search leads us to a file extensions explanation page, which states that the .do extension is typically a URL mapping scheme for compiled Java code.

### web.xml
Java web applications use a deployment descriptor file named web.xml to determine how URLs map to servlets, which URLs require authentication, and other information. This file is essential when we look for the implementations of any given functionality exposed by the web application.
Within the working directory, we see a WEB-INF folder, which is the Java's default configuration folder path where we can find the web.xml file. This file contains a number of servlet names to servlet classes as well as the servlet name to URL mappings. Information like this will become useful once we know exactly which class we are targeting, since it will tell us how to reach it.

### Java Path Finding

A natural question at this point might be: how do we know which Java process to target? In this case, we are fortunate as there is only one Java process running on our vulnerable machine. Some applications use multiple Java process instances though. In such cases, we can check any given process properties in Process Explorer by right-clicking on the process name and choosing Properties

### Interesting paths

```bash
C:\Program Files\ManageEngine\AppManager12\working # This path contains the web.xml file
C:\Program Files (x86)\ManageEngine\AppManager12\working\WEB-INF\lib # This path contains the .java files
```
### JD-GUI
Excellent tool for decompile .jar files
![JD-GUI](/assets/img/Pasted-image-20230703211001.png)

We first need to save the decompiled source code into human-readable .java files. JD-GUI allows us to do that via the File > Save All Sources menu. A tool for search strings Notepad++ which is already installed on our VM and could help us navigate this code base in a much easier way.
![JD-GUI-Decompiler](/assets/img/Pasted-image-20230703211144.png)

### Frontend Search

It is important to know that in a typical Java servlet, we can easily identify the HTTP request handler functions that handle each HTTP request type due to their constant and unique names.

These methods are named as follows:
-   _doGet_
-   _doPost_
-   _doPut_
-   _doDelete_
-   _doCopy_
-   _doOptions_

