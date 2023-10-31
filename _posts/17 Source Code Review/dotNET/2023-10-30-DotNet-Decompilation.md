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

![drag-and-drop-dnspy](/assets/img/Pasted-image-20231030230710.png)

To view the source code of this executable, we'll have to expand the test assembly navigation tree and select test.exe, dotnetapp, and then Program. According to the output, the decompilation process was successful.

![source-code-dnspy](/assets/img/Pasted-image-20231030230859.png)

#### Cross-References

When analyzing and debugging more complex applications, one of the most useful features of a decompiler is the ability to find cross-references to a particular variable or function. We can use cross-references to better understand the code logic. For example, we can monitor the execution flow statically or set strategic breakpoints to debug and inspect the target application. We can demonstrate the effectiveness of cross-references in this process with a simple example.

Let's suppose that while studying our DotNetNuke target application, we noticed a few Base64-encoded values in the HTTP requests captured by Burp Suite. Since we would like to better understand where these values are decoded and processed within our target application, we could make the assumption that any functions that handle Base64-encoded values contain the word "base64".

We'll follow this assumption and start searching for these functions in dnSpy. For a thorough analysis we should open all the .NET modules loaded by the web application in our decompiler. However, for the purpose of this exercise, we'll only open the main DNN module, C:\inetpub\wwwroot\dotnetnuke\bin\DotNetNuke.dll, and search for the term "base64" within method names as shown below:

![DNN-Method-Search](/assets/img/Pasted-image-20231031001854.png)

The search then throws the following results:
![DNN-Results-Search](/assets/img/Pasted-image-20231031002110.png)

> To retrieve such results we need to go to Options and unmark "Search in GAC assemblies" option
{: .prompt-tip }

By picking one of the functions and try to find its cross-references. We'll select the Base64UrlDecode function by right-clicking on it and selecting Analyze from the context menu.

![](/assets/img/Pasted-image-20231031024851.png){: .light .w-75 .shadow w='1212' h='668' }