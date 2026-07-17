### In-Depth: SCSI and the Linux Kernel

Whats SCSI?
Small Computer System Interface (SCSI) is a set of standards used to Physically connect and transfer data between computers and peripheral devices.

The traditional SCSI hardware setup is a host adapter linked with a chain of devices (disks) over an SCSI bus.
In simple words it means, that in older days when SCSI was being used to connect the physical data storage devices at that time the configuration was, that all the devices were connected to the `Host Adapter`, its a controller which sits inside the computer, Over an SCSI bus.

**Host Adapter** - The job of `Host Adapter` is to communicate between Computer and SCSI bus, it a translator between CPU and the storage devices.

**SCSI Bus** - Its a physical cable which is connected to the `Host adapter` in one end and on the other end it has Disks connected to it.

```
+-------------------+
|     Computer      |
|                   |
|  Linux Kernel     |
+---------+---------+
          |
          | PCI Card
          |
+---------v---------+
|    Host Adapter   |
+---------+---------+
          |
========== SCSI BUS ===========
     |        |         |
     |        |         |
+----v---+ +--v----+ +--v----+
| Disk 1 | | Disk2 | | Tape  |
+--------+ +-------+ +-------+
```

**The `Host Adapter` and the `Devices`(disks) each have an `SCSI ID`**, and there can be 8 or 16 IDs per bus, depending on the ISCI version.
```
Host Adapter  -> ID 7

Disk A        -> ID 0
Disk B        -> ID 1
Disk C        -> ID 2
Tape Drive    -> ID 3
```


**In todays time we don't get to see the physical connection of storage devices to the computer using SCSI Bus. As nowadays most of the use USB storage system.**
But the very important thing is that throughout the time - **SCSI has become a standard command language for storage devices.**
So by now SCSI commands has become a standard language which most storage devices like USB, DVD, SAS (Serial Attached SCSI) have different protocols but all uses the SCSI command language.

Definition: The SCSI Subsystem standardizes communication between different storage device types and kernel control commands.