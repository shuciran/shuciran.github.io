---
description: >-
  DotNet Modifying Assemblies
title:  DotNet Modifying Assemblies             # Add title here
date: 2023-10-31 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, DotNet Modifying Assemblies]                     # Change Templates to Writeup
tags: [scr, dotnet, modifying assemblies]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Assemblies Modification

We can arbitrarily modify assemblies, by using this technique to add debugging statements to a log file or alter an assembly's attributes in order to better debug our target application.

In order to demonstrate this technique, we will need to first [Decompile](https://shuciran.github.io/posts/DotNet-Decompilation/) our custom executable file and edit it using dnSpy. Let's right-click Program and choose Edit Class.

![Edit-Class-C](/assets/img/Pasted-image-20231031035520.png)

Then we'll change "Your answer was: " to "You said: " (Figure 40).

![Editing-Code](/assets/img/Pasted-image-20231031035656.png)

And finally, we'll click Compile, then File > Save All to overwrite the original version of the executable file.

![Save-All-File](/assets/img/Pasted-image-20231031035814.png)

![Overwrite-File](/assets/img/Pasted-image-20231031035947.png)

If we return to our command prompt and re-run test.exe, the second print statement is now "You said: "

![Modified Code](/assets/img/Pasted-image-20231031040233.png)
