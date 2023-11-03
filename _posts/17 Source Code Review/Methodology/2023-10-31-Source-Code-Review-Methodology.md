---
description: >-
   Source Code Review Methodology
title:  Source Code Review Methodology             # Add title here
date: 2023-10-31 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, Source Code Review Methodology]                     # Change Templates to Writeup
tags: [scr, methodology ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

As is always the case when we have access to the source code, we first like to just look around and get a feel for the application. How is it organized? Can we identify any coding style that can help us with string searches against the code base? Is there anything else that can help us streamline and minimize the amount of time we need to properly investigate our target?

There are many high-priority items to consider when performing manual source code analysis. This high-level list is presented in no particular order:

- After checking unauthenticated areas, focus on areas of the application that are likely to receive less attention (i.e. authenticated portions of the application).
- Investigate how sanitization of the user input is performed. Is it done using a trusted, open-source library, or is a custom solution in place?
- If the application uses a database, how are queries constructed? Does the application parameterize input or simply sanitize it?
- Inspect the logic for account creation or password reset/recovery routines. Can the functionality be subverted?
-Does the application interact with its operating system? If so, can we modify commands or inject new ones?
- Are there programming language-specific vulnerabilities?