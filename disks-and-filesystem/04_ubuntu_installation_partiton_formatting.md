### Ubuntu installation and the Formatting of Disk partitions
Lets say we are installation Ubuntu in an unallocated `200GB` space on the disk, and we tell the installer that we want 100GB to be allocated to `/` root and 100GB to be allocated to `/home`.

So below are the sequence of events that happens:
1. The installer creates 2 partitions of that unallocated 200GB space each of 100GB.
2. Then the installer formats the partitions with `ext4 filesystem`
    * mkfs.ext4 /dev/nvme0n1p6
    * mkfs.ext4 /dev/nvme0n1p7
3. Now each partitons contains its own `ext4` Filesystem, but both the partitions only contains `/` directory.
4. As part of user telling the installer to allocate 100GB to `/` root and 100GB to `/home`, so the installer did formatted partitions with `ext4` Filesystem, now it goes and mounts these 2 partitions to these 2 mount points:
    * `/`      nvme0n1p6
    * `/home`  nvme0n1p7
5. Now installer does remaining task of populating the `/` root directory with required directories like `etc, bin, var, ...` and the `/home` directory with user directories like `/home/ashwin` `/home/bob`.

