To install Empire on Kali Linux, we'll clone the project from the public GitHub repository with git clone, and run the install.sh script:

```
kali@kali:~$ cd /opt

kali@kali:/opt$ sudo git clone https://github.com/PowerShellEmpire/Empire.git
Cloning into 'Empire'...
remote: Enumerating objects: 12216, done.
remote: Total 12216 (delta 0), reused 0 (delta 0), pack-reused 12216
Receiving objects: 100% (12216/12216), 21.96 MiB | 3.22 MiB/s, done.
Resolving deltas: 100% (8312/8312), done.

kali@kali:/opt$ cd Empire/

kali@kali:/opt/Empire$ sudo ./setup/install.sh
...
```

Empire allows for collaboration between penetration testers across multiple servers using shared private keys and by extension, shared passwords. However, we are installing a single instance, so we'll press I at the password prompt to generate a random password.

```
...
 [>] Enter server negotiation password, enter for random generation:
```

With the framework installed, we can launch Empire with the aptly-named Python script, empire.

```
kali@kali:/opt/Empire$ sudo ./empire
...
================================================================
 [Empire]  Post-Exploitation Framework
================================================================
 [Version] 2.5 | [Web] https://github.com/empireProject/Empire
================================================================

   _______ .___  ___. .______    __  .______       _______
  |   ____||   \/   | |   _  \  |  | |   _  \     |   ____|
  |  |__   |  \  /  | |  |_)  | |  | |  |_)  |    |  |__
  |   __|  |  |\/|  | |   ___/  |  | |      /     |   __|
  |  |____ |  |  |  | |  |      |  | |  |\  \----.|  |____
  |_______||__|  |__| | _|      |__| | _| `._____||_______|


       285 modules currently loaded

       0 listeners currently active

       0 agents currently active


(Empire) >
```
