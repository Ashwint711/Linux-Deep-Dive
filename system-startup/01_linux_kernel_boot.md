### How Linux Kernel Boots

**Here is the simplified view of the boot process**
1. The machines boot firmware (BIOS/UEFI) loads and runs the boot loader.
2. Boot loader finds the kernel image and loads it into the memory.
3. The kernel initializes Devices and Device Drivers.
4. Kernel mounts the root filesystem.
5. The kernel starts a program called `init` with process ID 1. This point is the `user` space start.
6. Somewhere in the `init` process it runs the login process which lets you log in to the system.

### 1. Boot Firmware - BIOS or UEFI
Boot firmware is a piece of software which sits inside a `flash chip` attached on the `motherboard`. 
When the power button is pressed on a linux server at that time the CPU only knows one thing and that is to start executing the code from the `flash chip`.
So the CPU starts executing the boot firmware (BIOS or UEFI). The job of boot firmware is to initilize the hardware like CPU, RAM, SSD, and other hardware devices. 
And the main job of `boot firmware` is to find and load `boot loader` into memory.

### 2. Boot Loader - GRUB
Boot Loader is the second part, boot loader like `GRUB` sits inside block storage like ssd. And its job is to load and start execution of linux kernel.
The boot loader software like `GRUB` contains its own code to talk to hardware devices like SSSD. But in initial stages where the complete `GRUB` isn't loaded, till the it uses `BIOS` to for e.g. get sector 5642 from ssd.

### 3. Kernel
Kernel initializes devices and device drivers. And kernel mounts `root` filesystem to `/` in memory.
Kernel also stars `init` process in older systems in new systems it is `systemd`.