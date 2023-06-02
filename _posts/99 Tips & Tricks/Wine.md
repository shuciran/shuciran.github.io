If Wine is not working correctly on Kali it might be due to the ~/.wine profile, follow this steps:

That has to do with your wine prefix. Delete the standard prefix (~/.wine) or create a new custom one with: 

```bash
mkdir -p ~/myapp/prefix export WINEPREFIX=$HOME/myapp/prefix export WINEARCH=win32 export WINEPATH=$HOME/myapp wineboot --init winetricks
```