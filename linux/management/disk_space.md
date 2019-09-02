# Disk space

**How should we get the disk space on our Linux System?**

## df
Usually, we make use of 'df' command to dump the system's disk condition. When we add 'T' parameters, we will get each filesystem's type.

```c
[johnsonz@sfu03 ~]$ df -Th
Filesystem           Type   Size  Used Avail Use% Mounted on
/dev/mapper/vg_sfu03-lv_root
                     ext4    50G   31G   16G  66% /
tmpfs                tmpfs  3.9G  776K  3.9G   1% /dev/shm
/dev/sda1            ext4   477M   39M  413M   9% /boot
/dev/mapper/vg_sfu03-lv_home
                     ext4   860G  647G  170G  80% /home
FNQDA019:/ap         nfs    197G   94G   94G  51% /ap
```

**Note: There are some excptions for filesystem, and they don't make use of the Disk Space. such as tmpfs**
And you can get the filesystem's type through 'cat /etc/fstab'

```c
[johnsonz@sfu03 ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Tue Nov 21 00:32:26 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/vg_sfu03-lv_root /                       ext4    defaults        1 1
UUID=3052126a-2ba2-4418-9515-20c8940d07eb /boot                   ext4    defaults        1 2
/dev/mapper/vg_sfu03-lv_home /home                   ext4    defaults        1 2
/dev/mapper/vg_sfu03-lv_swap swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
FNQDA019:/ap         /ap                   nfs     defaults        0 0

```
## Some expections for Filesystem

- tmpfs is a temporary file storage paradigm implemented in many Unix-like operating systems. It is intended to appear as a mounted file system, but data is stored in volatile memory instead of a persistent storage device.
- sysfs
- procfs
- nfs
- devpts: devpts is a virtual filesystem available in the Linux kernel since version 2.1.93 (April 1998). It is normally mounted at /dev/pts and contains solely device files which represent slaves to the multiplexing master located at /dev/ptmx
