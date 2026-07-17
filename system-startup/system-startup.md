### What it takes for a computer to start

#### Small flash memory chip
There are n number of different Hardware components connected/attached to the Motherboard. Like:
* CPU
* RAM
* PCIe slots
* NVMe connectors
* **SPI Flash Chip** <-- contains UEFI firmware

In the previous days the computers had a hardware called `ROM` which stands for `Read Only Memory`, in those days the Firmware was burned onto this chip and it was read only.

But in todays world each motherboard have `SPI Flash Memory Chip` which can be written to, this chip contains the Firmware (UEFI/BIOS) which is the first program to run in any computer.

Basic Input/Output System (BIOS) - This was older firmware

Unified Extensible Firmware Interface (UEFI) - It is modern replacemnt of BIOS firmware program

When we Power on the PC, it triggers UEFI and it is loaded into RAM and CPU starts executing it.

#### What does UEFI roughly does?
* initialize CPU
* initialize RAM
* initialize PCIe
* initialize USB
* initialize keyboard
* initialize graphics
* detect SSDs
* detect USB drives

Only after all of these work is done by the UEFI firmware it looks for the Operating System.
#### But where does the UEFI look for the OS?
This is where the EFI System Partition (ESP) comes into picture.

### What is EFI system partition (ESP)?
The ESP is like other is normal system partition with `FAT32` filesystem formatted on it.
Why `fat32` filesystem and not `ext4`, this is because the `UEFI` firmware only understands `fat32` filesystem.

#### What is inside EFI system partition (ESP)?
```
EFI/
    Boot/
        bootx64.efi
    Ubuntu/
        grubx64.efi                    { All of these .efi are Boot Loader executables}
    Microsoft/
        bootmgfw.efi
```

*The `.efi` files are executable programs*

**So the story is, if your `SPI flash chips` has UEFI firmware then after completing the initial task of initializing the components etc. Then it goes and searches for EFI system partition.**
**But if the firmware is BIOS then it looks for `MBR boot sector` which is also a partition on the disk.**

