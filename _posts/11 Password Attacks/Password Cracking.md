### John The Ripper

Once we've gained access to password hashes from a target system, we can begin a password cracking session, running in the background, as we continue our assessment. If any of the passwords are cracked, we could attempt to use those passwords on other systems to increase our control over the target network. This, like other penetration testing processes, is iterative and we will feed data back into earlier steps as we expand our control.

To demonstrate password cracking, we will again turn to John the Ripper as it supports dozens of password formats and is incredibly powerful and flexible.

Running john in pure brute force mode (attempting every possible character combination in a password) is as simple as passing the file name containing our password hashes on the command line along with the hashing format.

```
kali@kali:~$ cat hash.txt
WDAGUtilityAccount:0c509cca8bcd12a26acf0d1e508cb028
Offsec:2892d26cdf84d7a70e2eb3b9f05c425e

kali@kali:~$ sudo john hash.txt --format=NT
Using default input encoding: UTF-8
Rules/masks using ISO-8859-1
Loaded 2 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
```

In the above output, JTR recognizes the hash type correctly and sets out to crack it. A brute force attack such as this, however, will take a long time based on the speed of our system. As an alternative, we can use the --wordlist parameter and provide the path to a wordlist instead, which shortens the process time but promises less password coverage:

```
kali@kali:~$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT
```

If any passwords remain to be cracked, we can next try to apply JTR's word mangling rules with the --rules parameter:

```
kali@kali:~$ john --rules --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT
```

In order to crack Linux-based hashes with JTR, we will need to first use the unshadow utility to combine the passwd and shadow files from the compromised system.

```
kali@kali:~$ unshadow passwd-file.txt shadow-file.txt
victim:$6$fOS.xfbT$5c5vh3Zrk.88SbCWP1nrjgccgYvCC/x7SEcjSujtrvQfkO4pSWHaGxZojNy.vAqMGrBBNOb0P3pW1ybxm2OIT/:1003:1003:,,,:/home/victim:/bin/bash

kali@kali:~$ unshadow passwd-file.txt shadow-file.txt > unshadowed.txt
```

We can now run john, passing the wordlist and the unshadowed text file as arguments:

```
kali@kali:~$ john --rules --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s3cr3t           (victim)
1g 0:00:00:28 DONE (2019-08-20 15:42) 0.03559g/s 2497p/s 2497c/s 2497C/s 
...
```

Newer versions of John the Ripper are multi-threaded by default but older ones only use a single CPU core to perform the cracking actions. If you encounter an older version of JTR, it supports alternatives that can speed up the process. We could employ multiple CPU cores, or even multiple computers, to distribute the load and speed up the cracking process. The --fork option engages multiple processes to make use of more CPU cores on a single machine and --node splits the work across multiple machines.

For example, let's assume we have two machines, each with an 8-core CPU. On the first machine we would set the --fork=8 and --node=1-8/16 options, instructing John to create eight processes on this machine, split the supplied wordlist into sixteen equal parts, and process the first eight parts locally. On the second machine, we could use --fork=8 and --node=9-16 to assign eight processes to the second half of the wordlist. Dividing the work in this manner would provide an approximate 16x performance improvement.

Attackers can also pre-compute hashes for passwords (which can take a great deal of time) and store them in a massive database, or _rainbow table_,[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/password-attacks/leveraging-password-hashes/password-cracking#fn1) to make password cracking a simple table-lookup affair. This is a space-time tradeoff since these tables can consume an enormous amount of space (into the petabytes depending on password complexity), but the password "cracking" process itself (technically a lookup process) takes significantly less time.

While John the Ripper is a great tool for cracking password hashes, its speed is limited to the power of the CPUs dedicated to the task. In recent years, Graphic Processing Units (GPUs) have become incredibly powerful and are, of course, found in every computer with a display. High-end machines, like those used for video editing and gaming, ship with incredibly powerful GPUs. GPU-cracking tools like Hashcat[2](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/password-attacks/leveraging-password-hashes/password-cracking#fn2) leverage the power of both the CPU and the GPU to reach incredible password cracking speeds.

### Hashcat

Hashcat's options generally mirror those of John the Ripper and include features such as algorithm detection and password list mutation.

In this example, we will run hashcat to identify the type of hash (-j -m) and then we'll use a dictionary and the type of attack (-a) in this case dictionary attack:

```
C:\Users\Cracker\hashcat-4.2.1> hashcat64.exe -j -m hash.txt
....
C:\Users\Cracker\hashcat-4.2.1> hashcat64.exe -m 13100 -a 0 hash.txt rockyou.txt
```
Examples:
[[Scramble#^49c9c0]]
