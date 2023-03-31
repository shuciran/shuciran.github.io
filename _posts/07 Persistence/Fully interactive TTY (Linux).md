---
description: >-
  Fully Interactive TTY (Linux)
title: Fully Interactive TTY (Linux)              # Add title here
date: 2022-08-24 08:00:00 -0600                           # Change the date to match completion date
categories: [07 Persistence, Fully Interactive TTY (Linux)]                     # Change Templates to Writeup
tags: [persistence, interactive tty]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### Python
Once we get access to the victim machine we proceed to stabilize the shell:
First we need to Ctrl + Z the shell as follows:
```bash
tom@epsilon:/var/www/app$ ^Z
zsh: suspended  nc -lvnp 1235
```
Then type the following command (stty raw -echo; fg) to get the shell back but with a threatment that allows us to for example delete a character, we will receive a message “continued” after that we type **reset xterm** in order to continue with the shell that was previously backgrounded:
```bash
stty raw -echo; fg
[1]  + continued  nc -lvnp 1235
                               reset xterm
```
Afterwards we’ll see that the shell is still pretty akward to work with because its output is staggered, in order to fix this, python3 is always a great tool to do it by executing the following command:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Finally modify some environment variables to get the same shell as our attack machine, by first looking into the content of such variables:
```bash
echo $TERM
unknown
export TERM=xterm

echo $SHELL
/bin/zsh
export SHELL=/bin/bash
```
Examples:
[Epsilon](https://shuciran.github.io/posts/Epsilon/#fnref:fully-interactive-tty)
[[Antique#^267c9b]]

### Script /dev/null
First we run the following command:
```bash
script /dev/null -c bash
```
Then, we need to Ctrl + Z the shell as follows:
```bash
root@smtp:/home# ^Z
zsh: suspended  nc -lvnp 443
```
Then type the following command (stty raw -echo; fg) to get the shell back but with a threatment that allows us to for example delete a character, we will receive a message “continued” after that we type **reset xterm** in order to continue with the shell that was previously backgrounded:
```bash
stty raw -echo; fg
[1]  + continued  nc -lvnp 1235
                               reset xterm
```
Finally export the environment variables with commands:
```bash
root@smtp:/home# export TERM=xterm
root@smtp:/home# export SHELL=bash
```