---
description: >-
  XSS Hunting
title:  XSS Hunting             # Add title here
date: 2023-07-11 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, XSS Hunting]                     # Change Templates to Writeup
tags: [scr, methodology, xss hunting]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

We'll start our hunt for DOM-based XSS by searching for references to the document object. However, running a search for "document" will generate many false positives. Instead, we'll search for "document.write" and narrow or broaden the search as needed. We will use grep recursively with the -r command in the ~/packages directory that we created earlier. To limit the results we will also use the --include flag to only search for HTML files.

```bash
kali@kali:~/packages$ grep -r "document.write" ./ --include *.html
./lodash-3.9.3/perf/index.html:			document.write('<script src="' + ui.buildPath + '"><\/script>');
./lodash-3.9.3/perf/index.html:			document.write('<script src="' + ui.otherPath + '"><\/script>');
./lodash-3.9.3/perf/index.html:						document.write('<applet code="nano" archive="../vendor/benchmark.js/nano.jar"></applet>');
./lodash-3.9.3/test/underscore.html:			document.write(ui.urlParams.loader != 'none'
./lodash-3.9.3/test/index.html:				document.write('<script src="' + ui.buildPath + '"><\/script>');
./lodash-3.9.3/test/index.html:			document.write((ui.isForeign || ui.urlParams.loader == 'none')
./lodash-3.9.3/test/backbone.html:			document.write(ui.urlParams.loader != 'none'
```

The results of this search reveal four unique files that write directly to the document. We also find interesting keywords like "urlParams" in the ui object that potentially point to the use of user-provided data. Let's (randomly) inspect the /lodash-3.9.3/perf/index.html file.

The snippet shown in Listing 11 is part of the /lodash-3.9.3/perf/index.html file.

```bash
<script src="./asset/perf-ui.js"></script>
<script>
        document.write('<script src="' + ui.buildPath + '"><\/script>');
</script>
<script>
        var lodash = _.noConflict();
</script>
<script>
        document.write('<script src="' + ui.otherPath + '"><\/script>');
</script>
```

In Listing 11, we notice the use of the document.write function to load a script on the web page. The source of the script is set to the ui.otherPath and ui.buildPath variable. If this variable is user-controlled, we would have access to DOM-based XSS.

Although we don't know the origin of ui.buildPath and ui.otherPath, we can search the included files for clues. Let's start by determining how ui.buildPath is set with grep. We know that JavaScript variables are set with the "=" sign. However, we don't know if there is a space, tab, or any other delimiter between the "buildPath" and the "=" sign. We can use a regex with grep to compensate for this.

```bash
kali@kali:~/packages$ grep -r "buildPath[[:space:]]*=" ./ 
./lodash-3.9.3/test/asset/test-ui.js:  ui.buildPath = (function() {
./lodash-3.9.3/perf/asset/perf-ui.js:  ui.buildPath = (function() {
```

The search revealed two files: asset/perf-ui.js and asset/test-ui.js. Listing 11 shows that ./asset/perf-ui.js is loaded into the HTML page that is being targeted. Let's open the perf-ui.js file and navigate to the section where buildPath is set.

```bash
kali@kali:~/packages$ cat ./lodash-3.9.3/perf/asset/perf-ui.js
...
  /** The lodash build to load. */
  var build = (build = /build=([^&]+)/.exec(location.search)) && decodeURIComponent(build[1]);
...
  // The lodash build file path.
  ui.buildPath = (function() {
    var result;
    switch (build) {
      case 'lodash-compat':     result = 'lodash.compat.min.js'; break;
      case 'lodash-custom-dev': result = 'lodash.custom.js'; break;
      case 'lodash-custom':     result = 'lodash.custom.min.js'; break;
      case null:                build  = 'lodash-modern';
      case 'lodash-modern':     result = 'lodash.min.js'; break;
      default:                  return build;
    }
    return basePath + result;
  }());
...
```

The ui.buildPath is set near the bottom of the file. A switch returns the value of the build variable by default if no other condition is true. The build variable is set near the beginning of the file and is obtained from location.search (the query string) and the value of the query string is parsed using regex. The regex looks for "build=" in the query string and extracts the value. We do not find any other sanitization of the build variable in the code. At this point, we should have a path to DOM XSS through the "build" query parameter!