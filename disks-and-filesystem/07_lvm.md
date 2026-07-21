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