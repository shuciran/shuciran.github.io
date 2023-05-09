---
description: >-
  Python Virtual Environment
title: Python Virtual Environment            # Add title here
date: 2022-12-15 08:00:00 -0600                           # Change the date to match completion date
categories: [05 Utilities, Python Virtual Environment]                     # Change Templates to Writeup
tags: [utilities, python, virtual environment]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

For python2 and Kali check this article:
[pyenv installation](https://www.kali.org/docs/general-use/using-eol-python-versions/)

### PyEnv 

Install dependencies:
```bash
sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git
```
Installation of pyenv:
```bash
curl https://pyenv.run | bash
```

If we are using ZSH then we will now add the proper lines to our .zshrc:
```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init --path)"\nfi' >> ~/.zshrc
```
To save the changes for such environment variables we can execute the following commands:
```bash
kali@kali:~$ exec $SHELL
kali@kali:~$ pyenv
```
We can now install Python 2 and set it as our default Python version:
```bash
kali@kali:~$ pyenv install 2.7.18
Downloading Python-2.7.18.tar.xz...
-> https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tar.xz
Installing Python-2.7.18...
Installed Python-2.7.18 to /home/kali/.pyenv/versions/2.7.18
kali@kali:~$ pyenv global 2.7.18
kali@kali:~$
kali@kali:~$ pyenv versions
  system
* 2.7.18 (set by /home/kali/.pyenv/version)
kali@kali:~$
kali@kali:~$ python
Python 2.7.18 (default, Apr 20 2020, 20:30:41)
[GCC 9.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

### Venv
Specify the python version
```bash
virtualenv -p /usr/bin/python2.7 venv
```

Activate the virtual environment:
```bash
source venv/bin/activate
```

Exit the venv command:
```bash
deactivate
```