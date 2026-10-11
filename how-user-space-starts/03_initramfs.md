### The Initial RAM Filesystem

Before diving into the `initramfs`, let's first understand how normal storage mechanism works in a Linux Machine.  

For example when a program makes a `write(fd, buffer, 4096)` system call, the CPU doesn't directly goes and writes to the block device. As Linux Knerel doesn't know how to directly talk to a SATA HDD or SSD directly. It depends on device specific drivers for that.

#### 1. SATA - HDD
```
Process
    ↓
VFS/Filesystem
    ↓
Linux Block Layer
    ↓
SATA/AHCI Driver
    ↓
SATA Controller
    ↓    
SATA Cable
    ↓
HDD
```

The SATA Controller is responsible for communicating over the SATA Interface. Linux has a driver for that controller, such as the AHCI driver.

So when Linux wants:
```
Read block 5000
```
the request eventually reaches to the controller, which communicates with the HDD using the SATA protocol.

#### 2. NVMe
With NVMe the architecture is different:
```
Process
    ↓
VFS/Filesystem
    ↓
Linux Block Layer
    ↓
NVMe Driver
    ↓
PCIe
    ↓
NVMe SSD
```

The NVMe SSD itself contains controller logic.

#### 3. RAID System

```
                 CPU
                  │
                  ↓
          RAID controller
             /    |    \
            ↓     ↓     ↓
          HDD1   HDD2   HDD3
```

The Operating System may see this:
```
/dev/sda
```
even though physically there are 3 disks.

The RAID Controller handles things like:
* communicates with disks
* distrubuting data across disks
* rebuilding failed disks
* caching reads/writes


#### So whats INITRAMFS

See when the system boots i.e; BIOS/UEFI firmware uses their code to scan the disk and find boot loader such as GRUB. And start boot loader, boot loader then then goes to the partition which have the kernel and then start kernel. Now further kernel initializes device drivers etc. and most importantly kernel needs to mount root filesystem into the memory.
But filesystem sits on the storage disk which can be SATA HDD or NVMe SSD or RAID storage. So then kernel needs storage device specific drivers to communicate with the actual storage hardware and read the filesystem from disk and mount it. But the problem is those device drivers are also stored on the same disk from which kernel is trying to mount filesystem. So it creates chicken-egg problem.

*To mount filesystem from storage device, kernel needs to read disk and to read it kernel depends on storage device drivers. But to use storage device drivers kernel needs to read the disk.*

To solve this we need some intermediary and here comes **initramfs**; The idea is somewhat similar to finding boot loader. The boot firmwares like BIOS/UEFI knows where they can find boot loader. Just like that **Linux puts the drivers needed to find and mount the real root filesystem somewhere that the kernel can access before the real root filesystem exists.** ***That "somewhere" is initramfs.***

So now the story goes like: BIOS/UEFI finds and loads boot loader (grub), boot loader then finds and loads Kernel And **initramfs**, and it starts kernel. Now kernel has the access to **initramfs** which boot loader already loaded into memory.
For a brief period of time `/` in memory represents the `initramfs`. Now with `initramfs` in memory kernel can locate and use the device drivers and with device drivers kernel is able to read the block device and it finally mounts the Real `root filesystem` to `/`.

Conceptually:
```
RAM
└── initramfs
       │
       │ loads driver
       ↓
   storage device
       │
       ↓
  /dev/sda2
       │
       ↓
real root filesystem
```

#### Complete startup flow:
```
             BIOS/UEFI
                 │
                 ↓
              GRUB
                 │
          ┌──────┴──────┐
          ↓             ↓
       Kernel        initramfs
          │             │
          └──────┬──────┘
                 ↓
        Kernel starts
                 │
                 ↓
       temporary RAM /
                 │
                 ↓
        initramfs /init
                 │
                 ↓
        load storage drivers
                 │
                 ↓
        find real root disk
                 │
                 ↓
       mount real filesystem
                 │
                 ↓
        switch to real /
                 │
                 ↓
         real init/systemd
                 │
                 ↓
          normal Linux
```