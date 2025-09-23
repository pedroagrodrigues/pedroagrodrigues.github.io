---
title: Increasing LVM Size on Linux
date: 2025-09-22 12:00:00
categories: [Linux, LVM]
tags: [Linux,LVM,Storage,Terminal]
---


Increasing disk space on a Linux machine may become necessary due to growing storage requirements, data expansion, application needs, file system limitations, or performance considerations. Expanding storage ensures sufficient capacity, accommodates future changes or updates, and prevents potential issues or performance degradation.

For this example, we’ll be using a **QEMU** virtual machine, but the steps apply to most Debian-based systems.


## Step 1 - Check current storage

First, check how much space is currently available using `df -h`:

```bash
    root@server:~# df -h
    Filesystem                         Size  Used Avail Use% Mounted on
    udev                               967M     0  967M   0% /dev
    tmpfs                              198M  564K  197M   1% /run
    /dev/mapper/server--vg-root  6.1G  2.1G  3.7G  37% /
    tmpfs                              986M     0  986M   0% /dev/shm
    tmpfs                              5.0M     0  5.0M   0% /run/lock
    /dev/sda1                          470M   87M  359M  20% /boot
    /dev/mapper/server--vg-var   2.3G  2.2G     0 100% /var
    /dev/mapper/server--vg-home   21G   72K   20G   1% /home
    /dev/mapper/server--vg-tmp   467M   19K  438M   1% /tmp
    tmpfs                              198M     0  198M   0% /run/user/1001
```

Here, `/var` is at 100% usage.

Now list all available disks with `fdisk -l`:

```bash
    root@server:~# fdisk -l
    Disk /dev/sda: 32 GiB, 34359738368 bytes, 67108864 sectors
    Disk model: QEMU HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x1dde1ea9

    Device     Boot   Start      End  Sectors  Size Id Type
    /dev/sda1  *       2048   999423   997376  487M 83 Linux
    /dev/sda2       1001470 67106815 66105346 31.5G  5 Extended
    /dev/sda5       1001472 67106815 66105344 31.5G 8e Linux LVM


    Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
    Disk model: QEMU HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/server--vg-root: 6.23 GiB, 6685720576 bytes, 13058048 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/server--vg-var: 2.38 GiB, 2558525440 bytes, 4997120 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/server--vg-swap_1: 976 MiB, 1023410176 bytes, 1998848 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/server--vg-tmp: 492 MiB, 515899392 bytes, 1007616 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/server--vg-home: 21.43 GiB, 23005757440 bytes, 44933120 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
```
This shows the existing 32 GB disk (/dev/sda) and the new 100 GB disk (/dev/sdb) that was added to the machine.

## Step 2 - Partitioning the new disk

Since `/dev/sdb` has no partitions, we’ll create one using `fdisk /dev/sdb`:

```bash
    root@server:~# fdisk /dev/sdb

    Welcome to fdisk (util-linux 2.36.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0x0fbdf1f1.
```
In this case we are creating a partition while leaving everything as default: `n p enter enter enter wq`

```bash
    Command (m for help): n
    Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1):
    First sector (2048-209715199, default 2048):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199):

    Created a new partition 1 of type 'Linux' and of size 100 GiB.
```
To see the new partition you can type `p`:

```bash
    Command (m for help): p
    Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
    Disk model: QEMU HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x0fbdf1f1

    Device     Boot Start       End   Sectors  Size Id Type
    /dev/sdb1        2048 209715199 209713152  100G 83 Linux
```

If everything is ready go ahead and write the partition and quit `fdisk`: 

```bash
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.      
```
You should now see the new partition `/dev/sdb1` if you run `fdisk -l` again:

```bash
Device     Boot Start       End   Sectors  Size Id Type
/dev/sdb1        2048 209715199 209713152  100G 83 Linux
```

Now let’s create a new physical device with the newly partitioned disk using `pvcreate`:

```shell
    pvcreate /dev/sdb1
```
Output:
```bash
    Physical volume "/dev/sdb1" successfully created
```
To display just run:

```bash
    pvdisplay
```
Output:

```bash
    --- Physical volume ---
    PV Name               /dev/sda5
    VG Name               server-vg
    PV Size               31.52 GiB / not usable 2.00 MiB
    Allocatable           yes
    PE Size               4.00 MiB
    Total PE              8069
    Free PE               13
    Allocated PE          8056
    PV UUID               FdNY35-hfqY-JG3g-B0D3-Yeld-wMRT-9K7UcE

    "/dev/sdb1" is a new physical volume of "<100.00 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sdb1
    VG Name
    PV Size               <100.00 GiB
    Allocatable           NO
    PE Size               0
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               qZbxlj-fScv-IjAC-RWj9-Cisw-LD8t-2TTTEx
```

## Step 3 - Check current volume group

Thereafter, let’s go ahead and check the current volume groups using `vgdisplay`:

```shell
    root@server:~# vgdisplay
    --- Volume group ---
    VG Name               server-vg
    System ID
    Format                lvm2
    Metadata Areas        1
    Metadata Sequence No  6
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                5
    Open LV               5
    Max PV                0
    Cur PV                1
    Act PV                1
    VG Size               <31.52 GiB
    PE Size               4.00 MiB
    Total PE              8069
    Alloc PE / Size       8056 / <31.47 GiB
    Free  PE / Size       13 / 52.00 MiB
    VG UUID               gWgysA-P62Y-BD3x-Mw1V-MeqU-oX40-LdM4JB
  ```

## Step 4 - Extend the current volume group.

Now we can extend it using `vgextend` and verify it using `vgdisplay`:

```shell
    root@server:~# vgextend ubuntu1604-vg /dev/sdb1
    Volume group "server-vg" successfully extended

    root@server:~# vgdisplay
    --- Volume group ---
    VG Name               server-vg
    System ID
    Format                lvm2
    Metadata Areas        2
    Metadata Sequence No  7
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                5
    Open LV               5
    Max PV                0
    Cur PV                2
    Act PV                2
    VG Size               <131.52 GiB
    PE Size               4.00 MiB
    Total PE              33668
    Alloc PE / Size       8056 / <31.47 GiB
    Free  PE / Size       25612 / <100.05 GiB
    VG UUID               gWgysA-P62Y-BD3x-Mw1V-MeqU-oX40-LdM4JB
```

After that, let's go ahead and check the logical volumes using `lvdisplay`:

```shell
    root@server:~# lvdisplay
    ---- Logical volume ---
    LV Path                /dev/server-vg/root
    LV Name                root
    VG Name                server-vg
    LV UUID                QBdSa5-Ol5p-0511-5dzY-7Erh-ZvxD-e4rPFl
    LV Write Access        read/write
    LV Creation host, time server, 2023-04-04 10:55:57 +0100
    LV Status              available
    # open                 1
    LV Size                <6.23 GiB
    Current LE             1594
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           254:0

    --- Logical volume ---
    LV Path                /dev/server-vg/var
    LV Name                var
    VG Name                server-vg
    LV UUID                E14kW7-dnny-7q1g-1wmu-EAMR-lkBs-psWmX8
    LV Write Access        read/write
    LV Creation host, time server, 2023-04-04 10:55:58 +0100
    LV Status              available
    # open                 1
    LV Size                2.38 GiB
    Current LE             610
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           254:1

    --- Logical volume ---
    LV Path                /dev/server-vg/swap_1
    LV Name                swap_1
    VG Name                server-vg
    LV UUID                3M776F-gHjB-QR9C-21TM-Dill-sszf-5gjyCa
    LV Write Access        read/write
    LV Creation host, time server, 2023-04-04 10:55:58 +0100
    LV Status              available
    # open                 2
    LV Size                976.00 MiB
    Current LE             244
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           254:2

    --- Logical volume ---
    LV Path                /dev/server-vg/tmp
    LV Name                tmp
    VG Name                server-vg
    LV UUID                Bc9VLb-inLP-Og1w-K27B-K5EZ-1jYu-HBufBB
    LV Write Access        read/write
    LV Creation host, time server, 2023-04-04 10:55:58 +0100
    LV Status              available
    # open                 1
    LV Size                492.00 MiB
    Current LE             123
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           254:3

    --- Logical volume ---
    LV Path                /dev/server-vg/home
    LV Name                home
    VG Name                server-vg
    LV UUID                um1BeP-GMLx-Ry3F-8KB1-psKU-37zW-9e82VJ
    LV Write Access        read/write
    LV Creation host, time server, 2023-04-04 10:55:58 +0100
    LV Status              available
    # open                 1
    LV Size                <21.43 GiB
    Current LE             5485
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           254:4
  
```
  
## Step 5 - Extend the volume.

In our case we are extending `var` volume by `32 GiB` using `lvextend`:

```shell

    root@server:~# lvextend --resizefs -L+32G /dev/server-vg/var
    Size of logical volume server-vg/var changed from 2.38 GiB (610 extents) to 34.38 GiB (8802 extents).
    Logical volume server-vg/var successfully resized.
    resize2fs 1.46.2 (28-Feb-2021)
    Filesystem at /dev/mapper/server--vg-var is mounted on /var; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 5
    The filesystem on /dev/mapper/server--vg-var is now 9013248 (4k) blocks long.
```

### Step 6 - Allocate entire disk space to the volume

You can assign a percentage of the disk with the following command:

```shell 
root@server:~# lvextend --resizefs -l +100%FREE /dev/server-vg/var
```

### Step 7 - Final Check

Verify the expansion worked with `df -h`.
The `/var` partition should now reflect the new size.

### Conclusion

Expanding LVM storage in Linux is a straightforward process once you understand the workflow: add a new `disk → create a physical volume → extend the volume group → resize the logical volume and filesystem`. This flexibility is one of the main advantages of using LVM, as it allows you to scale storage dynamically without downtime or complex migrations.

By following the steps above, you can ensure your system adapts to growing storage needs and keeps running smoothly.


Source: ***[Harry Vasanth](https://harryvasanth.com/posts/proxmox-increase-disk-size/#check-current-storage)***

Originally posted under the licence [**CC BY 4.0**](https://creativecommons.org/licenses/by/4.0/).