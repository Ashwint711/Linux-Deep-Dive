## What does mounting a Filesystem actually means?
Before answering above question, lets first undertand what does it mean to a disk or one partition of disk to have a filesystem formatted onto it. For this example lets use `ext4` filesystem.
So when we run this command:
```
mkfs.ext4 /dev/nvme0n1p6
```
the mkfs utility creates `ext4` filesystem in the `nvme0n1p6` partition. What does it mean to format `ext4` onto a partition - It means that there are some rules and decision made by the creators of ext4 filesystem such as:
*How to know the information that describes the entire file system. Or simply where to store the metadata about entire disk?*
-> ***Superblock***
*How to store file metadata separately from file's name and actual content?*
-> ***Inode Table***

Now we understand what does it mean to format a filesystem (`ext4`) onto a partition means.

Now lets look at the original question and that is - *what does mounting a filesystem means?*
So we formatted filesystem onto disk partition so now the user processes will be able to create, open files on this disk partition?
**No**
Why? 
Because all these things in the partition - superblock, inode table, etc are just bytes in real sense. We need some interpretation of it or a layer which will present these things as a directory structure. 
And solving this problem means ***Mounting a Filesystem***.

So in case of `ext4` filesystem the Linux kernel mounts it to `/` starting point of the Linux Directory structure.

### But where does this directory hierarchy lives?
The Linux Directory Hierarchy is stored in the disk, on disk it contains something like:
```
Root Inode

Directory entries

etc
bin
home
var
tmp
```
**If the linux directory hierarchy already is stored/lives in the disk partition then why do we need to mount the filesystem anywhere?**
That is because the CPU cannot work directly with the *bytes on an SSD*.

Suppose the SSD contains
```
Sector 1024

0x45
0xAA
0x10
...
```

The kernel doesn't expose raw sectors to applications. In simple words, the kernel doesn't expose raw bytes to applications. So there is need of an interpreter which will read the raw sectors and will present us something simple.

 So the `ext4 driver` interprets those bytes.
 It reads
 * the superblock
 * inode table
 * directory entries
 and creates kernel objects.

#### What is krernel object in this context?
On the ssd the directories arent stored as
```
Directory
---------
Name: home
Contains:
    ashwin
    alice
```
but they are stored as raw bytes, so for filesystem like ext4 we have ext4 filesystem driver which is the one who knows how these bytes are organized and it give us structure like above.
But everytime someone tries to access a directory the ext4 interpreter will need to interfere and translate raw bytes into meaningful directory structure. This would be very slow, so to solve this the kernel stores these meaningful structures in the Memory as objects and these objects are called Kernel Objects.

#### What happens during mount?
Suppose the kernel mounts ext4 filesystem
```
mount /dev/nvme0n1p6 /
```
The *ext4 driver* roughly does these steps:
1. Read the superblock
2. Verify that this is an ext4 filesystem
3. Read the *root inode* - `inode #2 = root directory` (Inode which represents the root directory)
4. **Now the kernel creates an object in RAM.
    ```
    Root Directory

    Name: /

    inode number: 2

    filesystem: ext4
    ```
    This object is not written into SSD, it only exists into Memory.
    
Then suppose user does 
```
cd /home
```
kernel asks - Inside *root inode* is there an entry for `/home` directory?
The *ext4* driver read the directory data from disk and finds `/home` and tells kernel about it, so now the kernel knows that this directory exists and it creates another *kernel object* for `/home` directory in the RAM.
```
Directory

Name: home

inode: 157
```

#### Mount filesystem
```
mount -t type device_name    mount_point
mount -t ext4 /dev/nvme0n1p1 /
```

#### Unmount filesystem
```
umount /dev/nvmen1p1
```