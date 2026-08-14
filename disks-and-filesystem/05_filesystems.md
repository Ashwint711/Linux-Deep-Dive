### Filesystems

The last link between the kernel and user space for disks is typically the filesystem.
**The filesystems is like a database which provides the structure to transform a Block Device into sophisticated hierarchy of files and directories that users can understand.**

Filesystems are traditionally implemented in the kernel but with the development of *9P* from *Plan 9* (Protocol) has inspired the development of User Space Filesystem. 
The *filesystem in user space (FUSE)* feature allows user space filesystem in Linux.

**Virtual Filesystem(VFS)** - The VFS abstraction layer completes the filesystem implementation. Much as the SCSI subsystem standardizes the communication between different storage devices and the kernel control commands, VFS ensures that all the filesystem implementations support a standard interface so that user-space applications access files and directories in the same manner.

```
User-space Applications
        |
Virtual File System
        |
File System (ext2, ext3, ext4)
```

#### Filesystem Types

* *The Fourth Extended Filesystem (ext4)* - This is the default filesystem in Linux.
* *B-tree Filesystem (btrfs)* - This is new filesystem native to Linux, designed to scale beyond the capabilities of ext4.


#### Creating a Filesystem
After creating partition we are ready to create or more precisely format a filesystem on that disk partition. Just like we do the disk partition operation in the user-spce, we can create a filesystem in the user-space only and do not need to switch to kernel mode. 
But how? The answer is that the user-spce processes have 2 ways to talk to the disk:
* Going through kernel - this path include the user space talking to filesystem which speaks the language of SCSI subsystem, which a mediator interface (eg. libata) understands and translates to the language that the device driver that talks to that Hardware disk device. All this happens in the kernel.
* Direct *Raw* access to the disk.

So the user-spce process `mkfs` chooses second way. To format a disk partition with `Fourth Extended Filesystem (ext4)` filesystem we can run:
```
mkfs.ext4 /dev/nvme0n1p6
```

#### What actually happens inside the Partition (nvme0n1p1)?
Formatting *ext4* onto disk means organizing the disk partition with the rules of this specific filesystem. There are things that are required to create a functional filesystem, so next are the things that the creators of *extended filesystem* have decided to be written on to the disk. In short here are the components of this filesystem:

#### Superblock 
This is the most important component of the *ext* series filesystem. Superblock is a reserved block on the disk with a specific fixed size which stores information/metadata about the disk like:
* Filesystem Geometry - Block size (eg. 4KB), total block count, total inode count, and reserved blocks for superuser.
* Current State - The total number of free blocks, free inodes

#### Inode Table
**In EXT4 filesystem the disk is divided into *Block Groups*, and each block group contains an *Inode Table*. This table is a array of fixed sized structures that store the metadata for every file and directory in that specific Block Group.**