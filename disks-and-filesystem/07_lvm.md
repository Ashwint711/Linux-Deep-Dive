### The Logical Volume Manager (LVM)

#### What problem are LVM is trying to solve?
Suppose i have a 1TB ssd and its divided into 3 partitions:
```
+------------------------------------------------+
| 300 GB (/)| 500 GB (/home) | 200 GB (/data)    |
+------------------------------------------------+
```
But imagine after a year the `/` root directory is almost full
```
/       290 GB used out of 300 GB
/home   150 GB used out of 500 GB
/data    20 GB used out of 200 GB
```
but the `/home` directory is mostly empty. So can the root directory partition borrows the space from home directory partition?
The answer is no because *partitions are fixed regions on the disk*.


Having disk partition is like having walls in a big apartment and those walls define room bounderies, that is okay but not very optimized as if one room is full and another have empty space, still it wont be able to because there is a physical wall in between.
What is we make that wall a logical wall, so instead of a physical wall the boundry will be painted on the ground and to increase the size of one room we can just redraw the line somewhere else.

Thats what LVM does with the given hardware storage. We give LVM our storage disks and its his job to provide us partitions managed by him, we do not care about how those partitions are being managed internally. For user the partition or logical-volume given by LVM can be treated as a normal partition, and we can install filesystem on it like we install on an actual disk partition.

#### But what happens under the hood?
```
VFS

ext4

Logical Volume(s)

Volume Group(s)

-------- LVM ---------

Physical volume(s)        { This level can mean real empty partitions of a disk or real multiple disk storages, 
                            so this level and below level kind of overlap}

SSD(s) - Hardware level
```

#### Physical Volume
A physical volume is a Block device (be it an entire ssd or partition of a ssd) that is initialized by LVM.
Physical Volume simply means that physical storage which LVM is aware of, so if i install a new ssd and I inform LVM about it then it becomes a Physical Volume.
Which implies that not every SSD on the system is a Physical Volume

Lets say I connect a new ssd(nvme1n1) and tell about this disk to LVM:
```
pvcreate /dev/nvme1n1
```

**A normal disk partition or a Loop device can be used as a Physical Volume. The command `pvcreate /dev/loop28` makes the normal block partition or file a lvm PV. LVM uses a concept called *Physical Extend (PE)*, a PE is a block inside a PV. Just like the ext4 filesystem divided the block device into block of 4kb like that LVM divides PV into blocks and they are called Physical Extent**

**The Physical Volume isn't actually divided into chunks called Physical Extents. When a partition is makde a Physical Volume so the LVM metadata is written onto the block device. Later when the PV becomes part of VG then the LVM treats the usable space as sequence of blocks (Physical Extents), the Physical Extent is the property of entire VG and not of a PV.**

This command writes some LVM metadata onto `/dev/nvme1n1` disk and tells `LVM` that you can use this block device.

#### Volume Group
*Its the grouping of Physical Volumes.* Suppose we have 2 block devices of 500GB each, but we need 2 devices with 700GB and 300GB.
So can we do that?
No

Why?
Because we don't have a single physical block device with 700GB storage. But we have combied 1000GB storage which is enough for our use case, so we need something which can take this physical block storage and give us our required block out of it and handles how to manage data between 2 block devices.

```
PV1 (500 GB)
      \
       \
        ---> VG (1000 GB)
       /
PV2 (500 GB)
```

**Logical volumes are just block devices, and they typically contain filesystems or swap signature, so we can think of the relationship between a volume group and its logical volume as similar to that of a disk and its partition**



### LVM Hands on

#### Block device
**Using file as a block device**
```
dd if=/dev/zero of=/home/disk1.img bs=1M count=15360
losetup --find
losetup /dev/loop28 /home/disk1.img
```

#### Making the block devices Physical Volume
```
pvcreate /dev/loop28
pvcreate /dev/loop29
```

```
ashwin@thinkpad /dev/mapper % sudo pvdisplay      
  --- Physical volume ---
  PV Name               /dev/loop28
  VG Name               myvg
  PV Size               5.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1279
  Free PE               0
  Allocated PE          1279
  PV UUID               r5gT42-sqwK-XZpR-IQ3j-Plft-bhmA-zGVwMN
   
  --- Physical volume ---
  PV Name               /dev/loop29
  VG Name               myvg
  PV Size               15.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              3839
  Free PE               254
  Allocated PE          3585
  PV UUID               k2Qdof-ZNCq-Nyhg-CqzB-ry83-0PWu-wbG4wf
```

#### Create Volume Group and include PVs
```
vgcreate myvg /dev/loop28
vgextend myvg /dev/loop29
```

```
ashwin@thinkpad /dev/mapper % sudo vgdisplay myvg 
  --- Volume group ---
  VG Name               myvg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               19.99 GiB
  PE Size               4.00 MiB
  Total PE              5118
  Alloc PE / Size       4864 / 19.00 GiB
  Free  PE / Size       254 / 1016.00 MiB
  VG UUID               0KvquO-iPHB-YYJM-2Ri6-ll1Y-CtIz-ab90vS
```

#### Carve Logical Volumes from VG

```
lvcreate --size 10g --type linear -n lv1 myvgs (The type linear is default)
lvcreate --size 9g --type linear -n lv2 myvgs
```

#### Format filesystem into LV and mount it
```
mkfs -t ext4 /dev/mapper/myvg-lv1

mount /dev/mapper/myvg-lv1 /mnt
```


### Device Mapper

Device Mapper is a kernel subsystem, which is a Translation Layer and it sits between Filesystem and Block device.

There are 2 jobs of Device Mapper:
#### 1. Create Device Nodes
Whenever a new LV is created it stores that pretty LV name under `/dev/mapper/vg-lv` and the actual device node name that Device Mapper works with under `/dev/dm-*`.

#### 2. Mapping Logical block to Physical Block
Imagine kernel without *Device Mapper*, something is trying to write bytes to logical block 500 of lv1, but lv1 is just a logical layout carved on a physical(underlying) disk. So here the kernel has to maintain which logical block maps to which physical(underlying) disk block.

This is the mapping problem from logical block to physical block(extent) that *Device Mapper* solves. Device Mapper maintains a mapping table.
Simple Example:
```
Logical Block Range        Physical Device

0 - 1279                  loop28

1280 - 5119               loop29
```


**This is why it's called a mapper**
It Maps
```
Virtual Block Device
```
to
```
real block device(s)
```