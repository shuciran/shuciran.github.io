---
description: >-
  DotNet Source Code Review
title:  DotNet Source Code Review             # Add title here
date: 2023-06-22 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, DotNet Source Code Review]                     # Change Templates to Writeup
tags: [scr, dotnet ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Manipulation of Assembly Attributes for Debugging
Debugging .NET web applications can sometimes be a bit tricky due to the optimizations that are applied to the executables at runtime. One of the ways these optimizations manifest themselves in a debugging session is by preventing us from setting breakpoints at arbitrary code lines. In other words, the debugger is unable to bind the breakpoints to the exact lines of code we would like to break at. As a consequence of this, in addition to not being able to break where we want, at times we are also not able to view the values of local variables that exist at that point. This can make debugging .NET applications harder than we would like.

Fortunately, there is a way to modify how a target executable is optimized at runtime.1 More specifically, most software will be compiled and released in the Release version, rather than Debug. As a consequence, one of the assembly attributes would look like this:

[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]
Listing 16 - Release versions of .NET assemblies are optimized at runtime

In order to enable a better debugging experience, i.e. to reduce the amount of optimization performed at runtime, we can change that attribute,2,3 to resemble the following:

[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default | DebuggableAttribute.DebuggingModes.DisableOptimizations | DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints | DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
Listing 17 - Specific assembly attributes can control the amount of optimization applied at runtime

As it so happens, this can be accomplished trivially using dnSpy. However, we need to make sure that we modify the correct assembly before we start debugging. In this instance, our target is the C:\inetpub\wwwroot\dotnetnuke\bin\DotNetNuke.dll file. It is important to note that once the IIS worker process starts, it will NOT load the assemblies from this directory. Rather it will make copies of all the required files for DNN to function and will load them from the following directory: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET Files\dotnetnuke\.

As always, before we do anything we should make a backup of the file(s) we intend to manipulate. We can then open the target assembly in dnSpy, right-click on its name in the Assembly Explorer and select the Edit Assembly Attributes (C#) option from the context menu (Figure 10). The same option can also be accessed through the Edit menu.

Figure 10: Accessing the Edit Assembly Attributes menu
Figure 10: Accessing the Edit Assembly Attributes menu
Clicking on that option opens an editor for the assembly attributes.

Figure 11: Assembly attributes
Figure 11: Assembly attributes
Here we need to replace the attribute we mentioned in Listing 16 (line 11) to the contents found in Listing 17.

Figure 12: Editing the assembly attributes
Figure 12: Editing the assembly attributes
Once we replace the relevant assembly attribute, we can just click on the Compile button, which will close the edit window. Finally, we'll save our edited assembly by clicking on the File > Save Module menu option, which presents us with the following dialog box:

Figure 13: Saving the edited assembly
Figure 13: Saving the edited assembly
We can accept the defaults and have the edited assembly overwrite the original. At this point we are ready to start using our dnSpy debugger.