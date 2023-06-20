##### MMLS
Tool which shows us the splits of the partitions in a system volume.
```bash
mmls filesystem.dump
```

![[Pasted image 20220813211159.png]]

##### FLS
Tool to list files and directory names it uses the output of mmls to work with:
```bash
fls -o 2048 filesystem.dump
```

##### TSK_RECOVER
Recover files from another files or directories:
```bash
tsk_recover -o 2048 filesystem.dump /path/to/save
```

##### DCFLDD
dd utility for forensics, it outputs the whole filesystem info as a binary:
```bash
dcfldd if=filesystem.dump > output.txt
```

##### BLKID
Show labels of a filesystem:
```bash
blkid filesystem.dump
```

Resources:
https://hackingpills.blogspot.com/2018/12/forensics-with-kali-linux-recovering.html?m=1

#### Mount Disk
First create an ext file with mke2 or mkfs
```bash
mke2fs -j /dev/hdb1 or
mkfs.ext3 -j /dev/hdb1

so now it is called hdb1 and you can mount it

Code:
mount /dev/hdb1 /mnt/hdb <-- be sure that dir exists
```