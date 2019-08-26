---
title: Create RAID 5
date: 2018-09-10 09:25:21
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "How to create raid"
---

Reference: https://www.tecmint.com/create-raid-5-in-linux/

## Install mdadm and verify drives

`yum install mdadm`

`mdadm -E /dev/sd[b-h]`

```
mdadm: No md superblock detected on /dev/sdc.

mdadm: No md superblock detected on /dev/sdb.

mdadm: No md superblock detected on /dev/sdd.

mdadm: No md superblock detected on /dev/sde.

mdadm: No md superblock detected on /dev/sdf.

mdadm: No md superblock detected on /dev/sdg.

mdadm: No md superblock detected on /dev/sdh.
```

## Partitioning the Disks for RAID

**Actually this is optional. If you want to partition first, you should make sure you're using gpt or others which support larger than 2T.**

##### Create /dev/sdb Partition

Please follow the below instructions to create partition on **/dev/sdb** drive.

`fdisk /dev/sdb`

<!--more-->

1. Press ‘**n**‘ for creating new partition.
2. Then choose ‘**P**‘ for Primary partition. Here we are choosing Primary because there is no partitions defined yet.
3. Then choose ‘**1**‘ to be the first partition. By default it will be **1**.
4. Here for cylinder size we don’t have to choose the specified size because we need the whole partition for RAID so just Press Enter two times to choose the default full size.
5. Next press ‘**p**‘ to print the created partition.
6. Change the Type, If we need to know the every available types Press ‘**L**‘.
7. Here, we are selecting ‘**fd**‘ as my type is RAID.
8. Next press ‘**p**‘ to print the defined partition.
9. Then again use ‘**p**‘ to print the changes what we have made.
10. Use ‘**w**‘ to write the changes.

After repeating these for all four disks, check for changes in all four drives sdb, sdc, sdd & sde.

`mdadm -E /dev/sd[b-e]`

```
/dev/sdb:
   MBR Magic : aa55
Partition[0] :   4294965247 sectors at         2048 (type fd)
/dev/sdc:
   MBR Magic : aa55
Partition[0] :   4294965247 sectors at         2048 (type fd)
/dev/sdd:
   MBR Magic : aa55
Partition[0] :   4294965247 sectors at         2048 (type fd)
/dev/sde:
   MBR Magic : aa55
Partition[0] :   4294965247 sectors at         2048 (type fd)
```

Now Check for the RAID blocks in newly created partitions. If no super-blocks detected, than we can move forward to create a new RAID 5 setup on these drives.

`mdadm --examine /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1`

```
mdadm: No md superblock detected on /dev/sdb1.

mdadm: No md superblock detected on /dev/sdc1.

mdadm: No md superblock detected on /dev/sdd1.

mdadm: No md superblock detected on /dev/sde1.
```

## Creating md device md0

`mdadm -C /dev/md0 -l 5 -n 4 /dev/sd[b-h]1`

```
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

After creating raid device, check and verify the RAID, devices included and RAID Level from the mdstat output.

`cat /proc/mdstat`

```
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdh[7] sdg[5] sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      46880191488 blocks super 1.2 level 5, 512k chunk, algorithm 2 [7/7] [UUUUUUU]
      
unused devices: <none>
```

## Watch process

`watch -n1 cat /proc/mdstat` refresh screen every 1 second.

## Verify

After creation of raid, Verify the raid devices using the following command.

`mdadm -E /dev/sd[b-h]`

```
/dev/sdb:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15627791024 (7451.91 GiB 8001.43 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : 61344c1d:281610e6:ce609f08:fd7d9ea7

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : e100b486 - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 0
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sdc:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15627791024 (7451.91 GiB 8001.43 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : 4daf07e5:2f98d4e5:36b6870e:20eedf68

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : 6faa7706 - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 1
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sdd:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15627791024 (7451.91 GiB 8001.43 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : b86dfb24:06c99d6a:12798289:9da23e0f

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : 55c0dda1 - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 2
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sde:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15626731520 (7451.41 GiB 8000.89 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : 54a9a755:6b0d7954:9dbeb452:872b894e

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : 78b50168 - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 3
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sdf:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15626731520 (7451.41 GiB 8000.89 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : fb8c7bd7:8e42bb21:0d402531:6db963d4

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : 2c16298a - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 4
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sdg:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15626731520 (7451.41 GiB 8000.89 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : 241d579a:b71126c8:0e70c185:024f7ae3

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : f90f4e73 - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 5
   Array State : AAAAAAA ('A' == active, '.' == missing)
/dev/sdh:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 7f86552b:c499fe55:d4be9502:c329d39c
           Name : node7:0  (local to host node7)
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
   Raid Devices : 7

 Avail Dev Size : 15626731520 (7451.41 GiB 8000.89 GB)
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 15626730496 (7451.41 GiB 8000.89 GB)
    Data Offset : 262144 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : bd88463c:51eb855a:ed8d4bab:58e25197

    Update Time : Sun Sep  9 05:24:11 2018
       Checksum : 6c044dd - correct
         Events : 18

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 6
   Array State : AAAAAAA ('A' == active, '.' == missing)
```

 Next, verify the RAID array to assume that the devices which we’ve included in the RAID level are running and started to re-sync.

`mdadm --detail /dev/md0`

```
/dev/md0:
        Version : 1.2
  Creation Time : Sat Sep  8 09:06:24 2018
     Raid Level : raid5
     Array Size : 46880191488 (44708.44 GiB 48005.32 GB)
  Used Dev Size : 7813365248 (7451.41 GiB 8000.89 GB)
   Raid Devices : 7
  Total Devices : 7
    Persistence : Superblock is persistent

    Update Time : Sun Sep  9 05:24:11 2018
          State : clean 
 Active Devices : 7
Working Devices : 7
 Failed Devices : 0
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : node7:0  (local to host node7)
           UUID : 7f86552b:c499fe55:d4be9502:c329d39c
         Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf
       5       8       96        5      active sync   /dev/sdg
       7       8      112        6      active sync   /dev/sdh
```



## Creating file system for md0

Create a file system for ‘**md0**‘ device using **ext4** before mounting.

`mkfs.ext4 -E lazy_itable_init=0,lazy_journal_init=0 /dev/md0`

```
mke2fs 1.41.9 (22-Aug-2009)
mkfs.ext4: Size of device /dev/md0 too big to be expressed in 32 bits
	using a blocksize of 4096.
```

Because `ext4` are still limited to 16 TB volumes, I decided to use `xfs`

`mkfs.xfs /dev/md0`

```
meta-data=/dev/md0               isize=256    agcount=44, agsize=268435328 blks
         =                       sectsz=4096  attr=2, projid32bit=0
data     =                       bsize=4096   blocks=11720047872, imaxpct=5
         =                       sunit=128    swidth=768 blks
naming   =version 2              bsize=4096   ascii-ci=0
log      =internal log           bsize=4096   blocks=521728, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

Now create a directory under ‘**/mnt**‘ then mount the created filesystem under **/mnt/raid5** and check the files under mount point, you will see **lost+found** directory. **If you use xfs filesystem, there's no lost+directory directory**

```
mkdir /mnt/raid5
mount /dev/md0 /mnt/raid5/
ls -l /mnt/raid5/
```

## Add entry in fstab

We need to add entry in **fstab**, else will not display our mount point after system reboot. To add an entry, we should edit the fstab file and append the following line as shown below. The mount point will differ according to your environment.

```
# vim /etc/fstab
/dev/md0                /mnt/raid5              xfs    defaults        0 0
```

Next, run ‘**mount -av**‘ command to check whether any errors in fstab entry.

```
mount: /dev/disk/by-id/scsi-35000cca0545b2b14-part1 already mounted on /boot
mount: /dev/md0 already mounted on /mnt/raid5
mount: proc already mounted on /proc
mount: devpts already mounted on /dev/pts
nothing was mounted
```

## Save Raid 5 Configuration

`mdadm --detail --scan --verbose >> /etc/mdadm.conf`

`cat /etc/mdadm.conf`

```
ARRAY /dev/md0 level=raid5 num-devices=7 metadata=1.2 name=node7:0 UUID=7f86552b:c499fe55:d4be9502:c329d39c
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf,/dev/sdg,/dev/sdh
```

