---
description: >-
    Git enumeration
title: Git enumeration                   # Add title here
date: 2023-02-07 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Git enumeration]                     # Change Templates to Writeup
tags: [git enumeration]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Githacker
Command to extract the whole git project:
```bash
githacker --url http://10.10.11.134/.git/ --output-folder results
``` 
Examples:
[a](/_posts/HTB/Linux/2%20Medium/2022-08-24-Epsilon.md)[^6edabc]

### Git
Command to list the commits under a git project (you should be under .git folder):
```bash
git log
```
After retrieving the hash for every commit, if we want to see the changes:
```bash
git show <hash-id>
```

Examples:
[[Epsilon#^a27371]]

[GITDUMPER](https://pentester.land/tutorials/2018/10/25/source-code-disclosure-via-exposed-git-folder.html)


