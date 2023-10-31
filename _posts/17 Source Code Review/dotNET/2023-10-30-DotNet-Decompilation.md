---
description: >-
  DotNet Decompilation
title:  DotNet Decompilation             # Add title here
date: 2023-10-30 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, DotNet Decompilation]                     # Change Templates to Writeup
tags: [scr, dotnet, decompilation]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### DnSpy

We use dnSpy and attempt to decompile an executable's code. We'll drag the test.exe file to the dnSpy window, which automatically triggers the decompilation process in dnSpy.

![drag-and-drop-dnspy](/assets/img/Pasted image 20231030230710.png)

To view the source code of this executable, we'll have to expand the test assembly navigation tree and select test.exe, dotnetapp, and then Program

![source-code-dnspy](/assets/img/Pasted image 20231030230859.png)