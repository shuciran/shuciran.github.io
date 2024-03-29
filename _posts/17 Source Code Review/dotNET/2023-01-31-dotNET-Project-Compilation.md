---
description: >-
  DotNet Project Compilation
title: DotNet Project Compilation             # Add title here
date: 2023-01-31 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, DotNet Project Compilation]                     # Change Templates to Writeup
tags: [scr, dotnet, compilation ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### C# Compilation Project

If we get a C# project not compiled, for example [SharpWSUS](https://github.com/nettitude/SharpWSUS) we can compile it with Visual Studio on a Windows machine, follow this steps:

Download the project onto your machine, you'll find a .sln file, click on it and Visual Studio will start automatically:
![Description](/assets/img/Pasted-image-20230125104122.png)

Go to the menu bar and choose "Release" and then click on the green play button:
![Description](/assets/img/Pasted-image-20230125104802.png)

If the following message appears, click on "Stop Debugging":
![Description](/assets/img/Pasted-image-20230125104842.png)

Finally go to the /bin/Release project's folder and you'll find the .exe file on it: 
![Description](/assets/img/Pasted-image-20230125104229.png)
Examples:
[Outdated](https://shuciran.github.io/posts/Outdated/#fnref:dotnet-compilation)