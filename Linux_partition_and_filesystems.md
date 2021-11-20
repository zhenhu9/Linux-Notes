# Linux Partitioning Concept

## Disk Terminology

**platter**, **head**, **track**, **cylinder**, **sector**

Data is commonly stored on magnetic or optical disk platters. The
platters are rotated (at high speeds). Data is read by heads, which
are very close to the surface of the platter, without touching it! The
heads are mounted on an arm (sometimes called a comb or a fork).

Hard disks are organized as a concentric stack of platters.

The data is stored on concentric circles on the surfaces known as
tracks.

Same tracks of different platters make a cylinder (all of the same
diameter).

Sections within each track are called sectors. A sector is the
smallest physical storage unit on a disk and typically it will hold
512 bytes of data.

The time it takes to position the head over a certain track is called
the seek time.

The maximum CHS values limited by legacy BIOS are:

- 1024 cylinders.
- 16 heads.
- 63 sectors per track.
- 512 bytes per sector.

**Notes**: 1024 x 16 x 63 x 512 bytes/sector = 528,482,304 bytes (528
million bytes or 504 MB)

Therefore, the largest hard disk drive size recognized directly by the
legacy BIOS is 504 MB.

Block is that A data storage area on a disk. A disk block is 512
bytes.

Disk controller is that a chip and its associated circuitry that
controls the disk drive.

Device driver is that a knernel module that controls a hardware or
virtual device.

Disk label is that the first sector of a disk that contains disk
geometry and partition information.

------

## Disk controller and disk types

- **ATA**

ATA Stands for Advanced Technology Attachment, is an interface that
connects hard drives, CD-ROM drives, and other drives. The first ATA
interface is now commonly referred to as PATA, which is short for
Parallel AT Attachment after the introduction of SATA.

IDE is a keyword which refers to the electrical specification of the
cables which connect ATA drives (like hard drives) to another device.
The drives use the ATA (Advanced Technology Attachment) interface.

SATA Stands for "Serial AT Attachment", and is the current leading
standard for harddrives and optical media.

- **SCSI**

A scsi controller allows more than two devices. When using SCSI (small
computer system interface), each device gets a unique scsi id. The
scsi controller also needs a scsi id, do not use this id for a
scsi-attached device.

Older 8-bit SCSI is now called narrow, whereas 16-bit is wide. When
the bus speeds was doubled to 10Mhz, this was known as fast SCSI.
Doubling to 20Mhz made it ultra SCSI. Take a look at
http://en.wikipedia.org/wiki/SCSI for more SCSI standards.

- **Block device**

Random access hard disk devices have an abstraction layer called block
device to enable formatting in fixed-size (usually 512 bytes) blocks.
Blocks can be accessed independent of access to other blocks.

**Notes**: A character device is a constant stream of characters,
being denoted by a c in ls -l. The ISO 9660 standard for cdrom uses a
2048 byte block size.

Old hard disks (and floppy disks) use cylinder-head-sector addressing
to access a sector on the disk. Most current disks use LBA (Logical
Block Addressing).

- **CD-ROM**

They are mainly indicated with sr0 and have properties such as RM of
1.

- **SSD**

A SSD (Solid State Drive) is a block device without moving parts.

- **loop devices**

The loop device is a block device that maps its data blocks not to a
physical device such as a hard disk or optical disk drive, but to the
blocks of a regular file in a filesystem or to another block device.

This can be useful for example to provide a block device for a
filesystem image stored in a file such as an ISO image,a disk image, a
file system, or a logical volume image. `ls -l /dev/loop*` to see the
loop block special device files.

------

## Device nameing

In Linux, partitions are represented by device files. A device file is
a file with type c ( for "character" devices, devices that do not use
the buffer cache) or b (for "block" devices, which go through the
buffer cache).  In Linux, all disks are represented as block devices
only.

### ATA (IDE) device naming

A typical PC has two IDE controllers, each of which can have two
drives connected to it. All ata drives on your system will start with
/dev/hd followed by a unit letter. The master hdd on the first ata
controller is /dev/hda, the slave is /dev/hdb. For the second
controller, the names of the devices are /dev/hdc and /dev/hdd.

**Table 1. IDE controller naming convention**

| controller  |  connection  | device name |
| ----------- | ------------ | ----------- |
|   ide0      |	 master      |  /dev/hda   |
|   ide0      |	 slave       |  /dev/hdb   |
|   ide1      |	 master      |  /dev/hdc   |
|   ide1      |	 slave       |  /dev/hdd   |

### SCSI device naming

scsi drives follow a similar scheme, but all start with /dev/sd. When
you run out of letters (after /dev/sdz), you can continue with
/dev/sdaa and /dev/sdab and so on. (We will see later on that lvm
volumes are commonly seen as /dev/md0, /dev/md1 etc.)

Below a sample of how scsi devices on a Linux can be named. Adding a
scsi disk or raid controller with a lower scsi address will change the
naming scheme (shifting the higher scsi addresses one letter further
in the alphabet).

**Table 2. SCSI device naming**

| device     |  scsi id  | device name |
| ---------- | --------- | ----------- |
|  disk 0    |    0      | /dev/sda    |
|  disk 1    |    1      | /dev/sdb    |
| raid controller 0 |   5    | /dev/sdc |
| raid controller 1 |   6    | /dev/sdd |

A modern Linux system will use `/dev/sd*` for scsi and sata devices,
and also for sd-cards, usb-sticks, (legacy) ATA/IDE devices and solid
state drives.

SCSI drives have ID numbers which go from 1 through 15. Lower SCSI ID
numbers are assigned lower-order letters. For example, if you have two
drives numbered 2 and 5, then #2 will be /dev/sda and #5 will be
/dev/sdb . If you remove either, all the higher numbered drives will
be renamed the next time you boot up.

If you have two SCSI controllers in your linux box, you will need to
examine the output of /bin/dmesg in order to see what name each drive
was assigned. If you remove one of two controllers, the remaining
controller might have all its drives renamed.

There are two work-arounds; both involve using a program to put a
label on each partition. The label is persistent even when the device
is physically moved. You then refer to the partition directly or
indirectly by label.

### Device numbers

The only important thing with a device file are its major and minor
device numbers, which are shown instead of the file size:

```
$ ls -l /dev/hda
```

**Table 3. Device file attributes**

| brw-rw---- |  1  |  root  |  disk  |  8, |  0  | Jul 18 18:57 | /dev/sda |
| ---------- | --- | ------ | ------ | --- | --- | ----------- | --- |
| permissions |  |  owner  |  group  |  major device number  | minor
device number  |  date  |  device name  |

When accessing a device file, the major number selects which device
driver is being called to perform the input/output operation. This
call is being done with the minor number as a parameter and it is
entirely up to the driver how the minor number is being interpreted.
The driver documentation usually describes how the driver uses minor
numbers.

**Table 4. Common Device naming**

|      Device Types     | Device Names   | Device Symlinks |
| --------------------- | ------------   | --------------- |
| floppy drive          | /dev/fd[0...]  |                 |
| IDE hard drive        | /dev/hd[a...]  |                 |
| SCSI/SATA/flash drive | /dev/sd[a...]  |                 |
| SCSI cdrom drive      | /dev/sr[0...]  | /dev/cdrom      |
| Serial port, COM      | ttyS[0...]     |                 |
| PS/2 mouse device     | psaux          | /dev/mouse      |
| Pseudo device, repeater data from GPM (mouse) daemon | gpmdata |

------

## Partitioning scheme

A modern typical pc normaly has two type of partitioning schemes,
depending on its BIOS and the boot mode.

**Table 5. Partitioning scheme**

| Partitioning scheme | BIOS type   | Boot mode |
| ------------------- | ----------- | --------- |
| MBR                 | legacy BIOS | legacy    |
| MBR                 | UEFI        | UEFI in CSM mode |
| GPT                 | UEFI        | Native UEFI |

**Notes**: Currently most PC systems that use UEFI also have a
so-called “Compatibility Support Module” (CSM) in the firmware, which
provides exactly the same interfaces to an operating system as a
classic PC BIOS, so that software written for the classic PC BIOS can
be used unchanged.

### MBR

The Master Boot Record is the traditional way of storing partition
information about a hard disk, along with some boot code. That is, the
Partition table is contained inside the MBR, which is stored in the
first sector (cylinder 0, head 0, sector 1 -- or, alternately, LBA 0)
of the hard drive.

**Table 5. MBR partition table contents**

| Size (bytes) | Description |
| ------------ | ----------- |
| 440  | MBR Bootstrap (flat binary executable code) |
| 4    | Optional "Unique Disk ID / Signature"       |
| 2    | Optional, reserved 0x0000    |
| 16   | First partition table entry  |
| 16   | Second partition table entry |
| 16   | Third partition table entry  |
| 16   | Fourth partition table entry |
| 2    | (0x55, 0xAA) "Valid bootsector" signature bytes |

1. The Bootstrap can be extended to 446 bytes if you omit the next 2
   optional fields: Disk ID and reserved.

2. The 4 byte "Unique Disk ID" is used by recent Linux and Windows
   systems to identify the drive. "Unique" in this case means that the
IDs of all the drives attached to a particular system are distinct.

3. The 2 byte reserved is usually 0x0000. 0x5A5A means read-only
   according to https://neosmart.net/wiki/mbr-boot-process/ .

### UEFI

(U)EFI or (Unified) Extensible Firmware Interface is a specification
for x86, x86-64, ARM, and Itanium platforms that defines a software
interface between the operating system and the platform firmware/BIOS.

**UEFI class 0-3 and CSM**

PCs are categorized as UEFI class 0, 1, 2, or 3. A class 0 machine is
a legacy system with a legacy BIOS; i.e. not a UEFI system at all.

A class 1 machine is a UEFI system that runs exclusively in
Compatibility Support Module (CSM) mode. CSM is a specification for
how UEFI firmware can emulate a legacy BIOS. UEFI firmware in CSM mode
loads legacy bootloaders. A class 1 UEFI system may not advertise UEFI
support at all, since it isn't exposed to the bootloader. It's only
UEFI "within" the BIOS.

A class 2 machine is a UEFI system that can launch UEFI applications
but also includes the option to run in CSM mode. The majority of
modern PCs are UEFI class 2 machines. Sometimes the choice to run UEFI
applications vs. CSM is a one-or-the-other setting in the BIOS
configuration, and other times the BIOS will decide which to use after
selecting the boot device and checking whether it has a legacy
bootloader or a UEFI application.

A class 3 machine is a UEFI system that does not support CSM. UEFI
class 3 machines only run UEFI applications and do not implement CSM
for backwards compatibility with legacy bootloaders.

### GPT

GPT stands for GUID (globally unique identifier device) Partition Table.

**Table 6. GPT partition contents**

| Addressing | description |
| ---------- | ----------- |
| LBA 0 | Protective Master Boot Record (PMBR). Holds a partition
pointing to GPT to avoid accidental overwrite by old programs. |
| LBA 1 | partition header, can be identified by 8 bytes magic "EFI
PART" (45h 46h 49h 20h 50h 41h 52h 54h) |
| LBA 2..33 | partition table entires |
|...data on disk...|   |
| LBA -33..-2 | mirror of partition table |
| LBA -1      | mirror of partition header on last addressable sector |

**LBA 0: Protective Master Boot Record**

This is kept untouched for backward compatibility to store a MBR
partition or a EFI partition which contian a boot loader. GPT-aware
firmware and utilities recognize the GPT and use that instead of the
MBR table.

### Primary, Extended and Logical partitions

A partition's geometry and size is usually defined by a starting and
ending cylinder (sometimes by sector). MBR partitions can be of type
primary (maximum four), extended (maximum one) or logical (contained
within the extended partition). Each partition has a type field that
contains a code.  GPT partitions only has primary partitions(maximum
128). This determines the computers operating system or the partitions
file system.

**Table 7. Partitioning's Limitation**

| Primary Partition | Extended parition | Logical Partition | Partition type | Boot mode |
| ----------------- | ----------------- | -------- | ------ | ------ |
| up to 4   | 0     | 0      | MBR      | legacy or UEFI in CSM boot |
| up to 3   | 1     | SCSI 15, IDE 64   | MBR    | legacy or UEFI in CSM boot |
| up to 128         | 0        | 0      | GPT      | Native UEFI boot |

### Partition Naming

We saw before that hard disk devices are named /dev/hdx or /dev/sdx
with x depending on the hardware configuration. Next is the partition
number, starting the count at 1. Hence the four (possible) primary
partitions are numbered 1 to 4. Logical partition counting always
starts at 5.

**Table 8. HD device partition naming**

| drive name | controller | drive number | partition type | partition number |
| ---------- | ---------- | ------------ | -------------- | ---- |
| /dev/hda1  |	  1       | 1 	       | primary 	| 1  |
| /dev/hda2  |	  1       | 1 	       | primary 	| 2  |
| /dev/hda3  | 	  1       | 1 	       | primary 	| 3  |
| /dev/hda4  | 	  1       | 1 	       | swap 		| NA |
| /dev/hdb1  | 	  1       | 2 	       | primary 	| 1  |
| /dev/hdb2  |	  1       | 2 	       | primary 	| 2  |
| /dev/hdb3  | 	  1       | 2 	       | primary 	| 3  |
| /dev/hdb4  | 	  1       | 2 	       | primary 	| 4  |

**Table 9. SCSI device partition naming**

| drive name | drive controller | drive number | partition type | partition number |
| ---------- | ----------- | ----------- | ----------- | ----------- |
| /dev/hdb1  |  1 	   |  2 	 | primary     |  1	     |
| /dev/hdb2  |  1 	   |  2 	 | extended    |  NA	     |
| /dev/hdb5  |  1 	   |  2 	 | logical     |  2	     |
| /dev/hdb6  |  1 	   |  2 	 | logical     |  3	     |

### Specific Partition

**Swap Partition**

Every process running on your computer is allocated a number of blocks
of RAM. These blocks are called pages. The set of in-memory pages
which will be referenced by the processor in the very near future is
called a "working set." Linux tries to predict these memory accesses
(assuming that recently used pages will be used again in the near
future) and keeps these pages in RAM if possible.

If you have too many processes running on a machine, the kernel will
try to free up RAM by writing pages to disk. This is what swap space
is for. It effectively increases the amount of memory you have
available. However, disk I/O is about a hundred times slower than
reading from and writing to RAM. Consider this emergency memory and
not extra memory.

------

## File System

A partition is labeled to host a certain kind of file system (not to
be confused with a volume label). Such a file system could be the
linux standard ext2 file system or linux swap space, or even foreign
file systems like (Microsoft) NTFS or (Sun) UFS. There is a numerical
code associated with each partition type. For example, the code for
native linux is 0x83 and linux swap is 0x82 . To see a list of
partition types and their codes, execute `/sbin/sfdisk -T`. To see a
list of filesystem types' overview, excute `man fs`.

**Table 10. File systems**

|  mark   |  file system |
| ------- | ------------ |
|  0x01   |  DOS FAT 12  |
|  0x04   |  DOS FAT 16 <32M    |
|  0x05   |  Extended partition |
|  0x06   |  DOS FAT 16  |
|  0x07   |  HPFS,NTFS,exFAT    |
|  0x0b   |  W95 FAT32   |
|  0x81   |  DR-DOS(Protected FAT),Linux/Minix |
|  0x82   |  swap        |
|  0x83   |  native linux       |

### FAT File System

There are several different versions of the FAT file system. Each
version was designed for a different size of storage media.

### FAT 12

FAT 12 was designed for floppy disks and can manage a maximum size of
16 megabytes because it uses 12 bits to address the clusters.

### FAT 16

FAT 16 was designed for early hard disks and could handle a maximum
size of 64K clusters * the cluster size. The larger the hard disk, the
larger the cluster size would be, which leads to large amounts of
"slack space" on the disk.

### FAT 32

FAT 32 was introduced to us by Windows95-B and Windows98. FAT32 solved
some of FAT's problems. No more 64K max clusters! FAT32 is slightly
misnamed, as the top 4 bits of the 32 bit cluster number are reserved,
and were never used. FAT 32 has certain system limitations. A file
stored on a FAT32 drive cannot be larger than 4 GB. In addition, the
entire FAT32 partition should be less than 8 TB in size.

### ExFAT

ExFAT is the filesystem used on SDXC cards, created by Microsoft. It
is basically FAT32 with actually 32 bits per FAT entry, with minor
extensions.

Microsoft has published the official specification at
https://docs.microsoft.com/en-us/windows/win32/fileio/exfat-specification.

### VFAT

VFAT is an extension to the FAT file system that has the ability to
use long filenames (up to 255 characters). First introduced by Windows
95, it uses a "kludge" whereby long filenames are marked with a
"volume label" attribute and filenames are subsequently stored in 11
byte chunks in sequential directory entries. (This is a bit of an
oversimplification, but close enough).

### HPFS

The HPFS was designed by IBM/Microsoft for IBMs new windowing system,
OS/2. It was designed to be fast, remove all the shortcomings of FAT,
support long filenames, small cluster sizes, remove fragmentation as
much as possible and support more attributes.

### NTFS

NTFS (New Technology File System) is Windows NT's native file system.
It is not only based on HPFS, but also supports security features such
as access control. Since Windows NT is entirely unicode, NTFS is a
unicode filesystem, with each character (e.g. in names) being 16-bits
instead of 8-bits.

- Support for files and disks of larger capacity that any other file
  systems can deal with.

- Support for extended file names and foreign characters from
  difficult languages.

- Severe decrease in system performance when «chkdsk» is used to check
  the local hard disk or an external disk.

- The standard system maintenance app «chkdsk» is notorious for its
  sluggishness.

- Increased security with the new file encryption feature.

- Much faster operation on drives smaller than 40 GB.

- Smaller file clusters.

- Compression is supported at the file system level for files,
  directories and drives to reduce disk space.

- User permissions for files and folders.

- File copies are «undone» if the interrupted cluster is cleaned.

- Small files are kept in the Master File Table at the beginning of
  the drive.

### ISO 9660

ISO 9660 is the standard file system for CD-ROMs. It is also widely
used on DVD and BD media and may as well be present on USB sticks or
hard disks. Its specifications are available for free under the name
ECMA-119.

### EXT

ext is an elaborate extension of the minix filesystem. It has been
completely superseded by the second version of the extended filesystem
(ext2) and has been removed from the kernel (in 2.1.21).

### EXT2

The Second Extended Filesystem (ext2fs) is a rewrite of the original
Extended Filesystem and as such, is also based around the concept of
"inodes."

- Introduced in 1993 by Remy Card. It was the first commercial-grade
  filesystem for Linux.

-  Does not supports Journaling.

- Fit for SD cards & USB drives since it has high performance and low
  writes (as journaling is not available). USB and SD storage are
limited with write cycles hence its best fit for them.

- Limits: Individual file size 16GB to 2TB. File system size 2TB to
  32TB.

### EXT3

Ext3 (the third extended filesystem) is the most commonly used
filesystem on Linux. It is basically an extension of ext2 to which a
journaling capability has been added.

- Introduced in 2001 by Stephen Tweedie. It was the most common
  filesystem in any Linux distro.

- Supports Journaling.

- Journaling keeps track of file changes which helps in fast recovery
  and reduce chances of data loss in case of a system crash.

- Limits: Individual file size 16GB to 2TB. File system size 4TB to
  32TB.

- Upgrading FS from ext2 to ext3 is an online process without
  downtime.

### EXT4

The fourth extended filesystem (ext4) is a journaling filesystem for
Linux.

- Introduced in 2008 by a team of developers. Its most the latest
  filesystem in ext family.

- Supports Journaling.

- Lots of new features introduced. Extents, Backward compatibility,
  Persistent pre-allocation, Delayed allocation, Unlimited number of
subdirectories, Journal checksum, Faster FS check, Transparent
encryption.

- Limits: Individual file size 16GB to 16TB. File system size up to
  1EB.

- Upgrading FS not needed. Due to backward compatibility, ext2, ext3
  can be directly mounted as ext4.

### XFS

XFS is Silicon Graphics' "Next Generation Journalled 64-Bit Filesystem
With Guaranteed Rate I/O" designed for IRIX based systems. XFS uses
the standard inodes, bitmaps and blocks, and is compatible with EFS
and NFS filesystems.

- Its 64-bit file system with a max file system size of 8EB.

- Supports metadata journaling which helps in faster data recovery in
  case of a system crash.

- It can be extended while mounted and active.

- It is internally partitioned into allocation groups. 4 different
  allocation groups available.

- Directory quotas help to limit quota over the directory tree.

- Quota journaling avoids quota consistency checks after the crash and
  hence quicker recovery.

- Extended attributes associated with each file. Those are additional
  name/value pairs.

- Online de-fragmentation is supported.

- It has native backup (xfsdump) & restore (xfsrestore) utilities.

### tmpfs

tmpfs is a filesystem whose contents reside in virtual memory. Since
the files on such filesystems typically reside in RAM, file access is
extremely fast. See tmpfs(5).

------

## partition flag

atvrecv - [GPT] This is a flag for Apple TV Recovery Partitions.

bios_grub - [GPT] The partition with this flag is used as the GRUB
BIOS partition. The partition usually uses about a megabytes of space
and is not needed by EFI-booting systems.

legacy_boot - [GPT] Some software (mostly legacy) recognize this flag
as indicating that the GPT partition may be bootable.

bls_boot - (MS-DOS, GPT) - Enable this to indicate that the selected
partition is a Linux Boot Loader Specification compatible /boot
partition.

boot - [Mac, MS-DOS, GPT, PC98] This flag indicates that the partition
is bootable. On MS-DOS partitioning tables, only one partition may
have this flag. However, other partitioning tables allow multiple
partitions to have this flag.

DIAG - [MS-DOS] The partition is used for recovery and diagnostics.

esp - [MS-DOS, GPT] This flag indicates an UEFI System Partition. GPT
uses this flag as an alias for "boot". The UEFI firmware stores files
on this partition.

chromeos_kernel - (GPT) this flag indicates a partition that can be
used with the Chrome OS bootloader and verified boot implementation.

hidden - [MS-DOS, PC98] Partitions with this flag are hidden from
Microsoft operating systems.

hp-service - [GPT] This the flag for HP Service Partitions. This
partition uses the FAT filesystem.

irst - [MS-DOS, GPT] Intel Rapid Start Technology partitions use this
flag.

lba - [MS-DOS] This flag is used by DOS, Windows 9x, and Windows ME
and tells the system to use Linear Mode (Logical Block Addressing).

LVM - [MS-DOS] This flag indicates to Linux that the partition is a
physical volume.

msftdata - [GPT] This flag is applied to Microsoft Basic Data
Partitions (BDP), such as NTFS and FAT filesystems including drive C:.
This flag can only be removed if the flags "boot" or "msftres" is
applied.

msftres - [GPT] Windows uses this flag to indicate a Microsoft
Reserved Partition (MSR). This is only used on GPT partitioning tables
and not MS-DOS (MBR) partitioning tables. Since GPT and the UEFI
specification do not allow hidden partitions (exceptions apply),
Microsoft Reserved Partitions are needed to replace hidden sectors
sometimes used by Windows. Windows (on GPT) needs this partition and
it must be before the primary and bootable partitions, but after the
EFI System Partition (ESP) and OEM service partitions.

PALO - [MS-DOS] The marked partition can use the Palo bootloader (used
by Linux/PA-RISC).

PREP - [MS-DOS, GPT] On PowerPC PReP and IBM RS6K/CHRP hardware, this
flag indicates that the marked partition supports PReP booting. The
PreP partition is not the same as the /boot/ directory or mountpoint.

raid - [MS-DOS] Linux recognizes this flag as a software RAID
partition.

root - [MAC] This flag is used to inform Linux which partition is the
root.

swap - [MAC] This is used to indicate that a swap partition is for a
Linux system.

------

## disklabel

- bsd
- loop (raw disk access)
- gpt
- mac (Apple Partition Map (APM))
- msdos (commonly called MBR)
- pc98
- sun

------

## Labels

In linux, hard drives are referred to as devices, and devices are
pseudo files in /dev. For example, the first partition of the second
lowest numbered SCSI drive is /dev/sdb1. If the drive referred to as
/dev/sda is removed from the chain, then the latter partition is
automatically renamed /dev/sda1 at reboot.

### Volume Labels

Volume labels make it possible for partitions to retain a consistent
name regardless of where they are connected, and regardless of
whatever else is connected. Labels are not mandatory for a linux
volume. Each can be a maximum of 16 characters long.

------

## Partitioning Utilities

**lsblk, parted, partprobe, fdisk, sfdisk, gparted**

**lsblk**

lsblk [options] [device ... ]

list block devices.

```
--topology	/* Output info about topology.
-f, --fs	/* Output info about filesystems.
-S, --scsi	/* Output info about SCSI devices only.
-a, --all	/* Also list empty devices and RAM disk devices.

/* Specify which output columns to print. Use --help to get a list of
all supported columns.
-o, --output list

-O, --output-all	/* Output all available columns.
```

Examples:

1. it's convinient to display block devices.

```
$ lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0    7:0    0     3G  1 loop /usr/lib/live/mount/rootfs/filesystem.squashfs
sda      8:0    0 465.8G  0 disk
sdb      8:16   1  28.9G  0 disk
├─sdb1   8:17   1   3.3G  0 part /usr/lib/live/mount/medium
└─sdb2   8:18   1   736K  0 part
sr0     11:0    1   403M  0 rom

Columns:
1. device name;
2. major and minor device numbers;
3. whether or not the device is removable (1 or 0);
4. size of the device;
5. whether or not the device is read only;
6. type of device (disk, partition, rom, loop, etc);
7. the device's mount point (if available).
```

2. Output info about block-device topology.

```
$ lsblk --topology

NAME   ALIGNMENT MIN-IO OPT-IO PHY-SEC LOG-SEC ROTA SCHED       RQ-SIZE  RA WSAME
loop0          0    512      0     512     512    1 mq-deadline     256 128    0B
sda            0   4096      0    4096     512    1 mq-deadline      64 128    0B
sdb            0    512      0     512     512    1 mq-deadline       2 128    0B
├─sdb1         0    512      0     512     512    1 mq-deadline       2 128    0B
└─sdb2         0    512      0     512     512    1 mq-deadline       2 128    0B
```

3. Display filesystems.

```
$ lsblk --fs

NAME   FSTYPE   FSVER            LABEL     UUID                    FSAVAIL FSUSE% MOUNTPOINT
loop0  squashfs 4.0                                                      0   100% /usr/lib/live/mount/rootfs/filesystem.squashfs
sda
sdb    iso9660  Joliet Extension Kali Live 2021-02-18-17-16-26-00
├─sdb1 iso9660  Joliet Extension Kali Live 2021-02-18-17-16-26-00        0   100% /usr/lib/live/mount/medium
└─sdb2 vfat     FAT12                      1366-8586
sr0    iso9660  Joliet Extension NERO12    2013-01-08-20-09-00-00
```

4. Display SCSI devices only.

```
$ lsblk --scsi
NAME HCTL       TYPE VENDOR   MODEL                 REV SERIAL                   TRAN
sda  0:0:0:0    disk ATA      HGST_HTS725050A7E630 A910 TF0500WJ108GYL           sata
sdb  4:0:0:0    disk Kingston DataTraveler_3.0     PMAP 60A44C3FAD9EB081298E00AF usb
sr0  5:0:0:0    rom  Slimtype eBAU108_7_L          YL0M 00_426050504950          usb
```

5. Display desired columns, partitions and filesystems uuid.

```
$ lsblk -o NAME,PARTUUID,UUID

NAME    PARTUUID                             UUID
loop0
sda
├─sda1  6aab50ec-4316-46b9-9985-78c4660aa497 8f1b3ff8-7b1c-4dc9-9ac5-cd28382870c6
├─sda2  a3af6d69-5a76-4886-9c77-1273f1a9c3f2
├─sda3  261f1fb1-e9bb-4409-877f-cc255ebe2891
├─sda4  8f13d23a-b67d-42fd-8971-c7e7a229407a
├─sda5  4b63ce63-43a1-45d2-88c2-a648a8ae621c
├─sda6  866b8b32-d489-4ab3-8966-07da37f7c7c2
├─sda7  7ba24fac-0c78-459e-af17-1aa2626daa85
├─sda8  8a058b65-04c2-4b9c-b59b-4b58b4b9f51e 4c2cc086-ba07-4e79-bb95-8e866eca67c7
├─sda9  4a586e17-ef39-4196-9b7a-3186bd5f2532 229dfc04-a7a7-4f17-9605-fdb2f9b77bd6
├─sda10 a98eb748-45a6-448e-b9db-a9c4706853bf cf4dcddb-fe84-42f5-9420-c46bf621771f
└─sda11 06ca2f23-1059-4673-8e75-f8005b20c503 8e1ec335-3d67-41f3-857a-7ec9a48c7fcb
sdb                                          2021-05-02-01-07-53-00
├─sdb1  67364e24-01                          2021-05-02-01-07-53-00
└─sdb2  67364e24-02                          608D-FB69
```

**Note**: `lsblk --help` to get column indicators.

6. Display desired columns, a very nice one.

```
$ lsblk -o NAME,SIZE,VENDOR,MODEL,TRAN,TYPE,FSTYPE,PARTLABEL,MOUNTPOINT

NAME      SIZE VENDOR   MODEL                TRAN   TYPE FSTYPE   PARTLABEL      MOUNTPOINT
loop0     3.1G                                      loop squashfs                /usr/lib/live/mount/rootfs/filesystem.squashfs
sda     465.8G ATA      HGST_HTS725050A7E630 sata   disk
├─sda1   1023M                                      part ext4     /boot
├─sda2      4G                                      part          /swap
├─sda3     35G                                      part          /usr
├─sda4     20G                                      part          /var
├─sda5     20G                                      part          /var/log
├─sda6     20G                                      part          /var/log/audit
├─sda7     80G                                      part          /var/tmp
├─sda8     80G                                      part xfs      /tmp
├─sda9     80G                                      part ext4     /home
├─sda10    60G                                      part ext4     /
└─sda11  65.8G                                      part ext4     freespace      /mnt
sdb      28.9G Kingston DataTraveler_3.0     usb    disk iso9660
├─sdb1    3.5G                                      part iso9660                 /usr/lib/live/mount/medium
└─sdb2    736K                                      part vfat
```

------

**parted**

parted [options] [device [command [options...]...]]

a partition manipulation program.

parted supports disklabel types, storage unit, partition types,
filesystem types, starting and ending locations of disk for making a
partition:

- disklabel types:
     aix, amiga, bsd, dvh, gpt, mac, msdos, pc98, sun, atari, loop.

- unit:
     "s" (sectors), "B" (bytes), "kB", "MB","MiB", "GB", "GiB", "TB",
"TiB", "%" (percentage of device size), "cyl" (cylinders), "chs"
(cylinders, heads, sectors), or "compact" (megabytes for input, and a
human-friendly form for output).

- partition types:

     primary, logical, extended.

- filesystem types:

     zfs, udf, btrfs, nilfs2, ext4, ext3, ext2, f2fs,fat32, fat16,
hfsx, hfs+, hfs, jfs, swsusp, linux-swap(v1), linux-swap(v0), ntfs,
reiserfs, freebsd-ufs, hp-ufs, sun-ufs, xfs, apfs2, apfs1, asfs,
amufs5, amufs4, amufs3, amufs2, amufs1, amufs0, amufs, affs7, affs6,
affs5, affs4, affs3, affs2, affs1, affs0, linux-swap, linux-swap(new),
linux-swap(old).

- starting and ending locations of disk for making a partition:

     use unit above. Negative values count from the end of the disk.
For example, -1s specifies exactly the last sector.

```
/* option

-l, --list	/* lists partition layout on all block devices.
-s, --script	/* never prompts for user intervention.

/* command

help [COMMAND]	/* print general help, or help on COMMAND.

unit UNIT		/* set the default unit to UNIT.

/* display the partition table, available devices, free space, all
found partitions, or a particular partition.
print [devices|free|list,all|NUMBER]

mklabel LABEL-TYPE	/* create a new disklabel (partition table).
mkpart PART-TYPE [FS-TYPE] START END	/* make a partition.
name NUMBER NAME	/* name partition NUMBER as NAME.

disk_set FLAG STATE	/* change the FLAG on selected device.
disk_toggle [FLAG]	/* toggle the state of FLAG on selected device.
set NUMBER FLAG STATE	/* change the FLAG on partition NUMBER.
toggle [NUMBER [FLAG]]	/* toggle the state of FLAG on partition NUMBER.

rm NUMBER		/* delete partition NUMBER.

resizepart NUMBER END	/* resize partition NUMBER.

rescue START END	/* rescue a lost partition near START and END.
```

Examples:

1. Display information about the specific disk.

```
$ sudo parted /dev/sda print

Model: ATA HGST HTS725050A7 (scsi)
Disk /dev/sda: 500GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary
```
**Note**: The physical sector size is 4096B it displaied above, so
this is a 4K device.

2. Delete all partitions on the disk to make new partitions.

```
$ sudo parted /dev/sda rm 1
```

3. Check if the partitions have been deleted.

```
$ sudo parted /dev/sda print

Model: ATA HGST HTS725050A7 (scsi)
Disk /dev/sda: 500GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start  End  Size  Type  File system  Flags
```

4. Create a new primary which size's nearly 100GiB. Cuz it's a 4K device,
   for device alignment, the starting location of disk would be 2048
sectors.

```
$ sudo parted /dev/sda mkpart primary ext4 2048s 100GiB
```

5. Create another primary partition.

```
$ sudo parted /dev/sda mkpart primary ext4 100GiB 200GiB
```

6. Create a extened to make logical partition.

```
$ sudo parted /dev/sda -- mkpart extended 200GiB -1s
```

**Note** the use of ‘--’, to prevent the following ‘-1s’ last-sector indicator from being interpreted as an invalid command-line option.

7. Create a logical partition.

```
$ sudo parted /dev/sda -- mkpart logical ext4 200GiB -1s

Warning: You requested a partition from 215GB to 500GB (sectors
419430400..976773167).
The closest location we can manage is 215GB to 500GB (sectors
419430401..976773167).
Is this still acceptable to you?
Yes/No? Yes
Warning: The resulting partition is not properly aligned for best
performance: 419430401s % 2048s != 0s
Ignore/Cancel? C
```

**Note**: Reserve 4k space to store contents of logical parition tabel
to solve aligment, so we should use more accurate numbers.

8. Get accurate unit and use it to partition again.

```
$ sudo parted /dev/sda print unit MiB print

Model: ATA HGST HTS725050A7 (scsi)
...
Number  Start      End        Size       Type      File system  Flags
 1      1.00MiB    102400MiB  102399MiB  primary
 2      102400MiB  204800MiB  102400MiB  primary
 3      204800MiB  476940MiB  272140MiB  extended               lba

$ sudo parted /dev/sda -- mkpart logical ext4 204801MiB -1s
```

**Note**: we reserve 1M to keep logical partiton table and alignment.

9. Check if the logical partition is properly aligned.

```
$ sudo parted /dev/sda align-check opt 5

5 aligned
```

10. delete all paritions on a command line.

```
$ sudo parted /dev/sda rm 5 rm 3 rm 2 rm 1
```


11. Create desired parititons on a command line.

```
$ sudo parted /dev/sda -- mkpart primary ext4 2048s 100GiB \
	mkpart extended 100GiB -1s \
	mkpart logical ext4 101GiB -1s
```

12. Check out the result of partitioning.

```
$ sudo parted /dev/sda print

Model: ATA HGST HTS725050A7 (scsi)
Disk /dev/sda: 500GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End    Size   Type      File system  Flags
 1      1049kB  107GB  107GB  primary
 2      107GB   215GB  107GB  primary
 3      215GB   500GB  285GB  extended               lba
 5      215GB   500GB  285GB  logical
```

13. Delete all partitions and make GPT patittions on a command line.

```
$ sudo parted /dev/sda rm 5 rm 3 rm 2 rm 1

$ sudo parted --script /dev/sda -- mklabel gpt \
mkpart /boot   2048s 1GiB \
mkpart /swap   1GiB  5GiB \
mkpart /usr    5GiB  40GiB \
mkpart /var    40GiB 60GiB \
mkpart /var/log  60GiB 80GiB \
mkpart /var/log/audit  80GiB 100GiB \
mkpart /var/tmp  100GiB 180GiB \
mkpart /tmp    180GiB 260GiB \
mkpart /home   260GiB 340GiB \
mkpart /       340GiB 400GiB \
mkpart freespace  400GiB -2048s
```

**Note**: GPT partition has a backup at the end of the disk.

14. Check out the result.

```
$ sudo parted /dev/sda print

Model: ATA HGST HTS725050A7 (scsi)
Disk /dev/sda: 500GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name            Flags
 1      1049kB  1074MB  1073MB  ext4         /boot
 2      1074MB  5369MB  4295MB               /swap
 3      5369MB  42.9GB  37.6GB               /usr
 4      42.9GB  64.4GB  21.5GB               /var
 5      64.4GB  85.9GB  21.5GB               /var/log
 6      85.9GB  107GB   21.5GB               /var/log/audit
 7      107GB   193GB   85.9GB               /var/tmp
 8      193GB   279GB   85.9GB               /tmp
 9      279GB   365GB   85.9GB               /home
10      365GB   429GB   64.4GB               /
11      429GB   500GB   70.6GB               freespace
```

15. Inform the kernel of partiton changes.

```
$ sudo partprobe
```

------

**partprobe**

partprobe [-d] [-s] [devices...]

inform the OS of partition table changes.

```
-d, --dry-run		/* Don't update the kernel.

-s, --summary	/* Show a summary of devices and their partitions.
```

Examples:

1. Inform the kernel.

```
$ sudo partprobe
```

2. Display a summary.

```
$ sudo partprobe -s

/dev/sda: gpt partitions 1 2 3 4 5 6 7 8 9 10 11
/dev/sdb: msdos partitions 1 2
```

------

**DD**

dd [option ...]

convert and copy a file.

```
if=FILE		/* read from FILE instead of stdin.

of=FILE		/* write to FILE instead of stdout.

bs=BYTES	/* read and write up to BYTES bytes at a time.

count=N		/* copy only N input blocks.

oflag=FLAGS	/* write as per the comma separated symbol list.

  sync		/* use synchronized I/O for data and metadata.

status=LEVEL	/* The LEVEL of information to print to stderr.

  'none'	/* Only display error messages.
  'noxfer'	/* Suppresses the final transfer statistics.
  'progress'	/* Display progress bar.

N and BYTES may be followed by the following multiplicative suffixes:
c=1, w=2, b=512, kB=1000, K=1024, MB=1000*1000, M=1024*1024, xM=M,
GB=1000*1000*1000, G=1024*1024*1024, and so on for T, P, E, Z, Y.
Binary prefixes can be used, too: KiB=K, MiB=M, and so on.  ```
```

Examples:

1. Burn a bootable iso image to a usb drive.

```
$ sudo dd if=kali.iso of=/dev/sdc bs=4M oflag=sync status=progress
```

**Note**: Before burning it, check out if the usb is unmounted. IF the
filesystem or partition support the size of the file, espacially when
it's large than 4.5GiB.

2. Likewise, but more simple.

```
$ sudo cp kali.iso /dev/sdc && sync
```

3. Backup MBR or GPT partition tables.

```
$ sudo dd if=/dev/sdb of=/tmp/MBR-partable.backup bs=512 count=1

$ sudo dd if=/dev/sdb of=/tmp/GPT-partable.backup bs=1M count=1
```

------

**fdisk**

fdisk [options] device

manipulate disk partition table.

fdisk is a dialog-driven program for creation and manipulation of
partition tables. It understands GPT, MBR, Sun, SGI and BSD partition
tables.

All partitioning is driven by device I/O limits (the topology) by
default. fdisk is able to optimize the disk layout for a 4K-sector
size and use an alignment offset on modern devices for MBR and GPT.
It is always a good idea to follow fdisk's defaults as the default
values (e.g., first and last partition sectors) and partition sizes
specified by the +/-<size>{M,G,...} notation are always aligned
according to the device properties.

CHS (Cylinder-Head-Sector) addressing is deprecated and not used by
default.

SIZES

The "last sector" dialog accepts partition size specified by number of
sectors or by +/-<size>{K,B,M,G,...} notation.

If the size is prefixed by '+' then it is interpreted as relative to
the partition first sector. If the size is prefixed by '-' then it
is interpreted as relative to the high limit (last available sector
for the partition).

In the case the size is specified in bytes than the number may be
followed by the multiplicative suffixes KiB=1024, MiB=1024*1024, and
so on for GiB, TiB, PiB, EiB, ZiB and YiB. The "iB" is optional, e.g.,
"K" has the same meaning as "KiB".

The relative sizes are always aligned according to device I/O limits.
The +/-<size>{K,B,M,G,...} notation is recommended.

For backward compatibility fdisk also accepts the suffixes KB=1000,
MB=1000*1000, and so on for GB, TB, PB, EB, ZB and YB. These 10^N
suffixes are deprecated.

```
-l, --list	/* list the partition table for the specified devices.

```

Example:

1. List partition tables for /dev/sda.

```
$ sudo fdisk -l /dev/sda

Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: HGST HTS725050A7
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 823B6DF0-8BB0-4282-9813-2D855B5EAF7A

Device         Start       End   Sectors  Size Type
/dev/sda1       2048   2097151   2095104 1023M EFI System
/dev/sda2    2097152  10485759   8388608    4G Linux swap
/dev/sda3   10485760  83886079  73400320   35G Linux filesystem
/dev/sda4   83886080 125829119  41943040   20G Linux filesystem
/dev/sda5  125829120 167772159  41943040   20G Linux filesystem
/dev/sda6  167772160 209715199  41943040   20G Linux filesystem
/dev/sda7  209715200 377487359 167772160   80G Linux filesystem
/dev/sda8  377487360 545259519 167772160   80G Linux filesystem
/dev/sda9  545259520 713031679 167772160   80G Linux filesystem
/dev/sda10 713031680 838860799 125829120   60G Linux filesystem
/dev/sda11 838860800 976771120 137910321 65.8G Linux filesystem
```

------

**sfdisk**

sfdisk is also a disk partition tool, but provide more features.

**gparted**

gparted is another disk partition tool with GUI interface.

------

## Formating an partition

mkfs and mkfs.* commands are designed to build file systems which you
required. There is a option -m to reserve the specified partition's
storage by percent. This prevent the file system from becoming filled.

`man fs` and `man ext2/ext3/ext4/xfs/...` to see the details of the
filesystems. `cat /proc/filesystems | grep -v nodev` to display
currently loaded file system drivers in the Linux kernel.

### Formating utilities

mkfs, mkfs.*, mke2fs, e2label, tune2fs, dumpe2fs, xfs-admin

**mkfs**

In actuality, mkfs is simply a front-end for the various filesystem
builders (mkfs.fstype) available under Linux. The filesystem-specific
builder is searched for via your PATH environment setting only.
Please see the filesystem-specific builder manual pages for further
details.

mkfs [options] [-t type] [fs-options] device [size]

mkfs.fstype device [options]

build a Linux filesystem.

```
-t, --type type		/* Specify the type of filesystem to be built.

/* Filesystem-specific options to be passed to the real filesys‐tem
builder.

fs-options

-U UUID		/* Set the UUID of the filesystem.
		/* May also be one of the following:
    clear		/* clear the filesystem UUID.
    random		/* generate a new randomly-generated UUID.
    time		/* generate a new time-based UUID.

-L new-volume-label	/* Set the volume label for the filesystem.

/* Specify the percentage of the filesystem blocks reserved for the
super-user. This avoids fragmentation, and allows root-owned daemons,
such as syslogd(8), to continue to function correctly after
non-privileged processes are prevented from writing to the filesystem.
The default percentage is 5%.

-m reserved-blocks-percentage

/* Force overwrite when an existing filesystem is detected on the
device.

-f
```

Examples:

1. Build a ext4 filesystem.

```
$ sudo mkfs -t ext4 /dev/sda10
```

2. Likewise.

```
$ sudo mkfs.ext4 /dev/sda10
or,
$ sudo mk2fs -t ext4 /dev/sda10
```

3. Build a xfs filesystem and set volume label to kali-home.

```
$ sudo mkfs.xfs -fL kali-home /dev/sda9
```

4. Check out the result.

```
$ lsblk --fs

NAME    FSTYPE   FSVER            LABEL     UUID                                 FSAVAIL FSUSE% MOUNTPOINT
loop0   squashfs 4.0                                                                   0   100% /usr/lib/live/mount/rootfs/filesystem.squashfs
sda
├─sda1  ext4     1.0                        8f1b3ff8-7b1c-4dc9-9ac5-cd28382870c6
├─sda2
├─sda3
├─sda4
├─sda5
├─sda6
├─sda7  xfs                       one       c665ed49-501c-4440-a25d-9b9822af9689
├─sda8  xfs                                 4c2cc086-ba07-4e79-bb95-8e866eca67c7
├─sda9  xfs                       kali-home f8662df5-5e0e-49f1-85db-9600fc6b0e24
├─sda10 ext4     1.0                        cf4dcddb-fe84-42f5-9420-c46bf621771f
└─sda11 ext4     1.0                        8e1ec335-3d67-41f3-857a-7ec9a48c7fcb
sdb     iso9660  Joliet Extension Kali Live 2021-05-02-01-07-53-00
├─sdb1  iso9660  Joliet Extension Kali Live 2021-05-02-01-07-53-00                     0   100% /usr/lib/live/mount/medium
└─sdb2  vfat     FAT12                      608D-FB69
```

------

**e2label**

e2label device [ volume-label ]

Change the label on an ext2/ext3/ext4 filesystem.

Examples:

1. set the label on a filesystem.

```
$ sudo e2label /dev/sda11 kali-free-e2
```

------

**tune2fs**

tune2fs [option ...] device

adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems.

```
-L
-m
-U UUID			/* all same as mke2fs options.
```

------

**dump2fs**

dumpe2fs [option ...] device

dump ext2/ext3/ext4 filesystem information.

------

**xfs_admin**

xfs_admin [option ...] device

change parameters of an XFS filesystem.

```
-L label
-U UUID			/* all same as mke2fs options.
```

------

## Swap Files

Normally, there are two ways to set up swap file. one is to create a
partition and make it a swap. another is to create a file by dd
command and make it a swap.

### Utilities about setting up wap space

mkswap, swapon, swapoff

**mkswap**

mkswap [options] device [size]

mkswap sets up a Linux swap area on a device or in a file.

```
-L, --label label	/* Specify a label for the device.
-f, --force		/* Go ahead even if the command is stupid.
-U, --uuid UUID		/* Specify the UUID. It's default.
```

**swapon, swapoff**

swapon [options] [specialfile...]
swapoff [-va] [specialfile...]

enable/disable devices and files for paging and swapping.

```
/* All devices marked as ``swap'' in /etc/fstab are made available,
except for those with the ``noauto'' option. Devices that are already
being used as swap are silently skipped.

-a, --all

-L label	/* Use the partition that has the specified label.

/* Specify the priority of the swap device. priority is a value
between -1 and 32767. Higher numbers indicate higher priority. See
swapon(2)  for a full description of swap priorities. Add pri=value to
the option field of /etc/fstab for use with swapon -a. When no
priority is defined, it defaults to -1.

-p, --priority priority

/* Display a definable table of swap areas. See the --help output for
a list of available columns.

--show[=column...]

--output-all		/* Output all available columns.

--bytes			/* Display swap size in bytes.

-U uuid			/* Use the partition that has the specified uuid.
```

Examples:

1. Make a swap partition and activate it.

```
$ mkswap -f /dev/target-partition
$ swapon /dev/target-partition
```

2. Create a 1M swap file and activate it.

```
$ dd if=/dev/zero of=/path/swap-filename bs=1024 count=1024
$ mkswap -f /path/swap-filename
$ swapon /path/swap-filename
```

You can create several swap space and mount them with different
priorities in /etc/fstab file. Then kernel will use the high priority
swap first. You also can use same priority on several swap space, then
the kernel will write to them much like a RAID, which is ideal in
terms of speed enhancement.

------

## Mounting filesystems

Once you've put a file system on a partition, you can mount it.
Mounting a file system makes it available for use, usually as a
directory. We say mounting a file system instead of mounting a
partition because we will see later that we can also mount file
systems that do not exists on partitions.

On all Unix systems, every file and every directory is part of one big
file tree. To access a file, you need to know the full path starting
from the root directory. When adding a file system to your computer,
you need to make it available somewhere in the file tree. The
directory where you make a file system available is called a mount
point.

**mount**

mount [option ...] -t fstype [-o options] device mountpoint

mount a filesystem.

Indicating the device and filesystem

Most devices are indicated by a filename (of a block special device),
like /dev/sda1, but there are other possibilities.

- LABEL=label	Human readable filesystem identifier. See also -L.

- UUID=uuid	Filesystem universally unique identifier.

- PARTLABEL=label	Human readable partition identifier. (GPT)

- PARTUUID=uuid	Partition universally unique identifier.

- ID=id		Hardware block device ID as generated by udevd.

COMMAND-LINE OPTIONS

```
-t, --types fstype
       The argument  following  the  -t  is  used  to  indicate  the
       filesystem  type.   The  filesystem types which are currently
       supported depend on the running kernel.   See  /proc/filesys‐
       tems  and  /lib/modules/$(uname  -r)/kernel/fs for a complete
       list of the filesystems.  The most  common  are  ext2,  ext3,
       ext4, xfs, btrfs, vfat, sysfs, proc, nfs and cifs.

-i, --internal-only
       Don't call the /sbin/mount.filesystem helper even if  it  ex‐
       ists.

-U, --uuid uuid
       Mount the partition that has the specified uuid.

-L, --label label
       Mount the partition that has the specified label.

-l, --show-labels
       Add  the labels in the mount output.  mount must have permis‐
       sion to read the disk device (e.g. be set-user-ID  root)  for
       this  to  work.   One  can set such a label for ext2, ext3 or
       ext4 using the e2label(8) utility, or for XFS  using  xfs_ad‐
       min(8), or for reiserfs using reiserfstune(8).

-o, --options opts
       use  the  specified  mount  options.   the opts argument is a
       comma-separated list.  for example:

              mount label=mydisk -o noatime,nodev,nosuid

       for more details, see the  filesystem-independent  mount  op‐
       tions and filesystem-specific mount options sections.


-r, --read-only
       Mount the filesystem read-only.  A synonym is -o ro.

       Note that, depending on the filesystem type, state and kernel
       behavior, the system may still write to the device.  For  ex‐
       ample,  ext3 and ext4 will replay the journal if the filesys‐
       tem is dirty.  To prevent this kind of write access, you  may
       want  to  mount an ext3 or ext4 filesystem with the ro,noload
       mount options or set the block  device  itself  to  read-only
       mode, see the blockdev(8) command.

-w, --rw, --read-write
       Mount the filesystem read/write.  Read-write  is  the  kernel
       default and the mount default is to try read-only if the pre‐
       vious mount syscall with read-write flags on  write-protected
       devices of filesystems failed.

       A synonym is -o rw.

-n, --no-mtab
       Mount without writing in /etc/mtab. This is necessary for
       example when /etc is on a read-only filesystem.
```

FILESYSTEM-INDEPENDENT MOUNT OPTIONS

```
defaults
       Use the default options: rw, suid, dev, exec, auto, nouser,
       and async.

       Note that the real set of all default mount  options  depends
       on the kernel and filesystem type.  See the beginning of this
       section for more details.

suid   Honor set-user-ID and set-group-ID bits or file  capabilities
       when executing programs from this filesystem.

nosuid Do  not honor set-user-ID and set-group-ID bits or file capa‐
       bilities when executing programs from this filesystem.

owner  Allow an ordinary user to mount the filesystem if  that  user
       is  the owner of the device.  This option implies the options
       nosuid and nodev (unless overridden by subsequent options, as
       in the option line owner,dev,suid).

user   Allow an ordinary user to mount the filesystem.  The name  of
       the mounting user is written to the mtab file (or to the pri‐
       vate libmount file in /run/mount on systems without a regular
       mtab)  so  that  this  same  user  can unmount the filesystem
       again.  This option implies the options noexec,  nosuid,  and
       nodev (unless overridden by subsequent options, as in the op‐
       tion line user,exec,dev,suid).

nouser Forbid an ordinary user to mount the filesystem.  This is the
       default; it does not imply any other options.

users  Allow  any  user to mount and to unmount the filesystem, even
       when some other ordinary user mounted it.   This  option  im‐
       plies  the options noexec, nosuid, and nodev (unless overrid‐
       den  by  subsequent  options,   as   in   the   option   line

dev    Interpret character or block special devices on the  filesys‐
       tem.

nodev  Do  not  interpret  character or block special devices on the
       filesystem.

exec   Permit execution of binaries.

noexec Do not permit direct execution of any binaries on the mounted
       filesystem.

async  All  I/O  to  the  filesystem  should be done asynchronously.
       (See also the sync option.)

sync   All I/O to the filesystem should be done  synchronously.   In
       the case of media with a limited number of write cycles (e.g.
       some flash drives), sync may cause life-cycle shortening.

atime  Do not use the noatime feature, so the inode access  time  is
       controlled  by kernel defaults.  See also the descriptions of
       the relatime and strictatime mount options.

noatime
       Do not update inode access times on this filesystem (e.g. for
       faster  access  on  the news spool to speed up news servers).
       This works for all inode types (directories too), so  it  im‐
       plies nodiratime.

auto   Can be mounted with the -a option.

noauto Can  only be mounted explicitly (i.e., the -a option will not
       cause the filesystem to be mounted).

_netdev
       The filesystem resides on a device that requires network  ac‐
       cess  (used  to  prevent  the system from attempting to mount
       these filesystems until the network has been enabled  on  the
       system).
```

------

**fstab**

/etc/fstab

static information about the filesystems.

The file fstab contains descriptive information about the filesystems
the system can mount. fstab is only read by programs, and not written;
it is the duty of the system administrator to properly create and
maintain this file.  The order of records in fstab is important
because fsck(8), mount(8), and umount(8) sequentially iterate through
fstab doing their thing.

each filesystem is described on a separate line. Fields on each line
are separated by tabs or spaces. Lines starting with '#' are comments.
Blank lines are ignored.


The following is a typical example of an fstab entry:

LABEL=t-home2	/home	ext4	defaults,auto_da_alloc	0  2

- The first field (fs_spec).

This field describes the block special device, remote filesystem or
filesystem image for loop device to be mounted or swap file or swap
partition to be enabled (source).

- The second field (fs_file).

This field describes the mount point (target) for the filesystem. For
swap partitions, this field should be specified as `none'. If the name
of the mount point contains spaces or tabs these can be escaped as
`\040' and '\011' respectively.

- The third field (fs_vfstype).

This field describes the type of the filesystem.

- The fourth field (fs_mntops).

This field describes the mount options associated with the filesystem.

- The fifth field (fs_freq).

This field is used by dump(8) to determine which filesystems need to be dumped. Defaults to zero (don't dump) if not present.

- The sixth field (fs_passno).

This field is used by fsck(8) to determine the order in which
filesystem checks are done at boot time. Defaults to zero (don't fsck)
if not present.

------

/proc/mounts

The kernel provides the info in /proc/mounts in file form, but
/proc/mounts does not exist as a file on any hard disk. Looking at
/proc/mounts is looking at information that comes directly from the
kernel.

/etc/mtab

The /etc/mtab file is not updated by the kernel, but is maintained by
the mount command. Do not edit /etc/mtab manually.

The support for regular classic /etc/mtab is completely disabled at
compile time by default, because on current Linux systems it is better
to make /etc/mtab a symlink to /proc/mounts instead.

------

## Loop devices

**losetup**

losetup [option ...] [loopdev]

set up and control loop devices.

**Note**: It's possible to create more independent loop devices for
the same backing file. This setup may be dangerous, can cause data
loss, corruption and overwrites. Use --nooverlap with --find during
setup to avoid this problem.

Options:

```
-a, --all		/* Show the status of all loop devices.
-l, --list		/* Show the status in column.

/* Specify the columns that are to be printed. Use --help to get a
list of all supported columns.

-O, --output column[,column] ...

/* Show the status of all loop devices associated with the given file.
-j, --associated file

/* Find the first unused loop device, If a file argument is present,
use the found device as loop device. Otherwise, just print its name.
-f, --find [file]

/* Display the name of the assigned loop device if the -f option and a
file argument are present.
--show

/* Check for conflicts between loop devices to avoid situation when the
same backing file is shared between more loop devices. If the file is
already used by another device then reuse the device rather than a new
one. The option makes sense only with --find.
-L, --nooverlap

-r, --read-only		/* Set up a read-only loop device.

/* Enable or disable direct I/O for the backing file.
--direct-io[=on|off]

-d, --detach loopdev...	/* Detach the specified loop device(s).
-D, --detach-all	/* Detach all loop devices.
```

Examples:

1. Create a img file.

```
$ sudo dd if=/dev/zero of=/mnt/test/test.img bs=1MiB count=20

20+0 records in
20+0 records out
20971520 bytes (21 MB, 20 MiB) copied, 0.0440482 s, 476 MB/s
```

2. Attach the specified file to the found loop device.

```
$ sudo losetup --find

/dev/loop2

$ sudo losetup /dev/loop2 /mnt/test/test.img
```

3. Likewise, but only one step.

```
$ sudo losetup --find --nooverlap --show /mnt/test/test.img

/dev/loop2
```

4. Formating the loop device.

```
$ sudo mkfs -t ext4 /dev/loop2
```

5. Mounting the device to the mountpoint.

```
$ sudo mkdir /mnt/Mpoint

$ sudo mount /dev/loop2 /mnt/Mpoint
```

6. Check out the result.

```
$ lsblk

NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
...
loop2     7:2    0    20M  0 loop /mnt/Mpoint
sda       8:0    0 465.8G  0 disk
├─sda1    8:1    0  1023M  0 part
├─sda2    8:2    0     4G  0 part
...
├─sda9    8:9    0    80G  0 part
├─sda10   8:10   0    60G  0 part
└─sda11   8:11   0  65.8G  0 part /mnt/test
```

------

## Other Utilities about filesystems

findmnt, df, du,

**findmnt**

findmnt [options] device|mountpoint

find a filesystem.

findmnt will list all mounted filesystems or search for a filesystem. The findmnt command is able to search in /etc/fstab, /etc/mtab or /proc/self/mountinfo. If device or mountpoint is not given, all filesystems are shown.

By default, findmnt will list all mounted filesystems in tree style.

```
-D, --df
       Imitate  the  output  of df(1).  This option is equivalent to
       -o SOURCE,FSTYPE,SIZE,USED,AVAIL,USE%,TARGET but excludes all
       pseudo filesystems.  Use --all to print all filesystems.

-o, --output list
       Define output columns.  See the --help output to get  a  list
       of  the  currently supported columns.  The TARGET column con‐
       tains tree formatting if the --list or --raw options are  not
       specified.

-e, --evaluate
        Convert  all tags (LABEL, UUID, PARTUUID or PARTLABEL) to the
        corresponding device names.

--pseudo
       Print only pseudo filesystems.

--real Print only real filesystems.

-f, --first-only
       Print the first matching filesystem only.

-i, --invert
       Invert the sense of matching.
```

Examples:

1. Normally, we just use this command to display the mount information.

```
$ findmnt

TARGET                                             SOURCE       FSTYPE          OPTIONS
/                                                  overlay      overlay         rw,noatime,lowerdir=/run/live/rootfs/filesystem.squashfs/,upperdir=/run/live/overlay/rw,workdir=/run/live/overlay/work
├─/sys                                             sysfs        sysfs           rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security                           securityfs   securityfs      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup                                 cgroup2      cgroup2         rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
│ ├─/usr/lib/live/mount/rootfs/filesystem.squashfs /dev/loop0   squashfs        ro,noatime
...
│ └─/usr/lib/live/mount/overlay                    tmpfs        tmpfs           rw,noatime,mode=755
├─/tmp                                             tmpfs        tmpfs           rw,nosuid,nodev,relatime
└─/mnt                                             /dev/sda11   ext4            ro,nosuid,noexec,relatime,sync
```

------

**df**

df [OPTION]... [FILE]...

report file system disk space usage.

```
-t, --type=TYPE
       limit listing to file systems of type TYPE

-h, --human-readable
       print sizes in powers of 1024 (e.g., 1023M)
```

Example:

1. Display file system disk space usage in human-readable.

```
$ df -Th

Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs          tmpfs     783M  1.5M  782M   1% /run
/dev/sdb1      iso9660   3.6G  3.6G     0 100% /run/live/medium
/dev/loop0     squashfs  3.2G  3.2G     0 100% /run/live/rootfs/filesystem.squashfs
tmpfs          tmpfs     3.9G  968M  2.9G  25% /run/live/overlay
overlay        overlay   3.9G  968M  2.9G  25% /
tmpfs          tmpfs     3.9G  108M  3.8G   3% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     3.9G  2.3M  3.9G   1% /tmp
tmpfs          tmpfs     783M  848K  783M   1% /run/user/1000
tmpfs          tmpfs     783M  4.0K  783M   1% /run/user/134
/dev/sda11     ext4       65G  724M   61G   2% /mnt
```

------

**du**

du [OPTION]... [FILE]...

estimate file space usage.

```
-h, --human-readable
       print sizes in human readable format (e.g., 1K 234M 2G)

-s, --summarize
       display only a total for each argument
```

Examples:

1. Display a toal of the desired directory in human readable size.

```
$ du -sh /boot

91M	/boot
```

------

**lsof**

lsof [options ...]

list open files.


------

**fuser**

fuser [options ...] name ...

identify processes using files or sockets.



------

## Reference

http://linux-training.be/index.php?nav=storage
https://docs.oracle.com/cd/E19253-01/817-5093/disksconcepts-35202/index.html
