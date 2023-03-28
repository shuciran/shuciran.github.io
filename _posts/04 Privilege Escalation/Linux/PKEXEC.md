## PwnKit (ly4k)

This [PwnKit](https://github.com/ly4k/PwnKit) contains a pretty good PwnKit binary to exploit PKEXEC. In order to exploit it, we need to download the PwnKit.c binary and compile from our Kali:

(This is to compile with x32 architecture if x64 is needed, delete "-m32")
```bash
gcc -m32 -shared PwnKit.c -o PwnKit -Wl,-e,entry -fPIC
```

Upload it to the victim machine and execute it, you can use python server.
