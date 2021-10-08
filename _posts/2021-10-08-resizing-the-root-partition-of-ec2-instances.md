## Scenario

The server only has 1 mounted disk & partition, it is out of space.

```
[ec2-user@ip-172-31-30-152 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        992M   56K  992M   1% /dev
tmpfs          1001M     0 1001M   0% /dev/shm
/dev/xvda1       99G   95G  4.2G  96% /
```

## Procedure

```
[root@ip-172-31-30-152 ~]# fdisk -l

Disk /dev/xvda: 161.1 GB, 161061273600 bytes, 314572800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00000000

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1               1   209715199   104857599+  ee  GPT
```
.
```
[root@ip-172-31-30-152 ~]# lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  100G  0 part /
```
.
```
[root@ip-172-31-30-152 ~]# growpart /dev/xvda 1
CHANGED: disk=/dev/xvda partition=1: start=4096 old: size=209711070,end=209715166 new: size=314568670,end=314572766
```
.
```
[root@ip-172-31-30-152 ~]# lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  150G  0 part /
```
.
```
[root@ip-172-31-30-152 ~]# resize2fs /dev/xvda1
resize2fs 1.42.12 (29-Aug-2014)
Filesystem at /dev/xvda1 is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 10
The filesystem on /dev/xvda1 is now 39321083 (4k) blocks long.
```
