## LVM

### What is LVM

LVM stands for Logical Volume Management. It is a system of managing
logical volumes, or filesystems, that is much more advanced and
flexible than the traditional method of partitioning a disk into one
or more segments and formatting that partition with a filesystem.

------

### Why use LVM?

For a long time I wondered why anyone would want to use LVM when you
can use gparted to resize and move partitions just fine. The answer is
that lvm can do these things better, and some nifty new things that
you just can't do otherwise. I will explain several tasks that lvm can
do and why it does so better than other tools, then how to do them.
First you should understand the basics of lvm.

------

### The Basics

There are 3 concepts that LVM manages:

- Volume Groups
- Physical Volumes
- Logical Volumes 

A Volume Group is a named collection of physical and logical volumes.
Typical systems only need one Volume Group to contain all of the
physical and logical volumes on the system, and I like to name mine
after the name of the machine. Physical Volumes correspond to disks;
they are block devices that provide the space to store logical
volumes. Logical volumes correspond to partitions: they hold a
filesystem. Unlike partitions though, logical volumes get names rather
than numbers, they can span across multiple disks, and do not have to
be physically contiguous.

### The Specifics

One of the biggest advantages LVM has is that most operations can be
done on the fly, while the system is running. Most operations that you
can do with gparted require that the partitions you are trying to
manipulate are not in use at the time, so you have to boot from the
livecd to perform them. You also often run into the limits of the
msdos partition table format with gparted, including only 4 primary
partitions, and all logical partitions must be contained within one
contiguous extended partition.

------

### Resizing Partitions

With gparted you can expand and shrink partitions, but only if they
are not in use. LVM can expand a partition while it is mounted, if the
filesystem used on it also supports that ( like the usual ext3/4 ).
When expanding a partition, gparted can only expand it into adjacent
free space, but LVM can use free space anywhere in the Volume Group,
even on another disk. When using gparted, this restriction often means
that you must move other partitions around to make space to expand
one, which is a very time consuming process that can result in massive
data loss if it fails or is interrupted ( power loss ).

------

### Moving Partitions

Moving partitions with gparted is usually only necessary in the first
place because of the requirement that partitions be physically
contiguous, so you probably won't ever need to do this with LVM. If
you do, unlike gparted, LVM can move a partition while it is in use,
and will not corrupt your data if it is interrupted. In the event that
your system crashes or loses power during the move, you can simply
restart it after rebooting and it will finish normally. When I got my
SSD drive, I simply plugged it in, booted it up, and asked lvm to move
my running root filesystem to the new drive in the background while I
continued working. Another reason you might want to move is to replace
an old disk with a new, larger one. You can migrate the system to the
new disk while using it, and then remove the old one later.

------

### Many Partitions

If you like to test various Linux distributions, or just different
version of Ubuntu, or both, you can quickly end up with quite a few
partitions. With conventional msdos partitions, this becomes
problematic due to its limitations. With LVM you can create as many
Logical Volumes as you wish, and it is usually quite easy since you
usually have plenty of free space left. Usually people allocate the
entire drive to one partition when they first install, but since
extending a partition is so easy with LVM, there is no reason to do
this. It is better to allocate only what you think you will need, and
leave the rest of the space free for future use. If you end up running
out of the initial allocation, adding more space to that volume is
just one command that completes immediately while the system is
running normally.

------

### Snapshots

This is something you simply can not do without LVM. It allows you to
freeze an existing Logical Volume in time, at any moment, even while
the system is running. You can continue to use the original volume
normally, but the snapshot volume appears to be an image of the
original, frozen in time at the moment you created it. You can use
this to get a consistent filesystem image to back up, without shutting
down the system. You can also use it to save the state of the system,
so that you can later return to that state if you mess things up. You
can even mount the snapshot volume and make changes to it, without
affecting the original.

------

## The Details

### So how do I start using LVM?

The alternate installer has the ability to set up and install to LVM,
and is the supported way of doing so. You can install the lvm2 package
on an existing system, or the desktop livecd and manually set it up,
and then install to it. This is what I will cover. (Ubuntu 12.10 has
since introduced LVM support from the installation live CD.)

First, you need a Physical Volume. Typically you start with a hard
disk, and create an LVM type partition on it. You can create one with
gparted or fdisk, and usually only want one partition to use the whole
disk, since LVM will handle subdividing it into Logical Volumes. In
gparted, you need to check the lvm flag when creating the partition,
and with fdisk, tag the type with code 8e.

Once you have your LVM partition, you need to initialize it as a
Physical Volume. Assuming this partition is /dev/sda1:

sudo pvcreate /dev/sda1

This writes the LVM header to the partition, which identifies it as a
Physical Volume, and sets up a small area to hold the metadata
describing everything about the Volume Group, and the the rest of the
partition as unused Physical Extents. After that, you need to create a
Volume Group named foo:

sudo vgcreate foo /dev/sda1

Now you have a Volume Group named foo. I suggest you change foo to a
name meaningful to you. foo contains only one Physical Volume. Now you
want to create a Logical Volume from some of the free space in foo:

 sudo lvcreate -n bar -L 5g foo 

This creates a Logical Volume named bar in Volume Group foo using 5 GB
of space. If you are installing, you probably want to create a Logical
Volume like this to use as a root filesystem, and one for swap, and
maybe one for /home. I currently have a Logical Volume for a Lucid
install, and one for a Maverick install, so that is what I named those
volumes. You can find the block device for this Logical Volume in
'/dev/foo/bar' or 'dev/mapper/foo-bar'.

You might also want to try the lvs and pvs commands, which list the
Logical Volumes and Physical Volumes respectively, and their more
detailed variants; lvdisplay and pvdisplay.

If you are doing this from the desktop livecd, once you have created
your Logical Volumes from the terminal, you can run the installer, and
use manual partitioning to select how to use each Logical Volume, and
then install.

------

### Resizing Partitions

You can extend a Logical Volume with:

 sudo lvextend -L +5g foo/bar 

This will add 5 GB to the bar Logical Volume in the foo Volume Group.
You can specify an absolute size if you want instead of a relative
size by omitting the leading +. The space is allocated from any free
space anywhere in the bar Volume Group. If you have multiple Physical
Volumes you can add the names of one or more of them to the end of the
command to limit which ones should be used to satisfy the request.

After extending the Logical Volume you need to expand the filesystem
to use the new space. For ext 3/4, you simply run:

 sudo resize2fs /dev/foo/bar 

------

### Moving Partitions

If you only have one Physical Volume then you probably will not ever
need to move, but if you add a new disk, you might want to. To move
the Logical Volume bar off of Physical Volume /dev/sda1, you run:

 sudo pvmove -n bar /dev/sda1 

If you omit the -n bar argument, then all Logical Volumes on the
/dev/sda1 Physical Volume will be moved. If you only have one other
Physical Volume, then that is where it will be moved to, or you can
add the name of one or more specific Physical Volumes that should be
used to satisfy the request, instead of any Physical Volume in the
Volume Group with free space. This process can be resumed safely if
interrupted by a crash or power failure, and can be done while the
Logical Volume(s) in question are in use. You can also add -b to
perform the move in the background and return immediately, or -i s to
have it print how much progress it has made every s seconds. If you
background the move, you can check its progress with the lvs command.

------

### Snapshots

When you create a snapshot, you create a new Logical Volume to act as
a clone of the original Logical Volume. The snapshot volume initially
does not use any space, but as changes are made to the original
volume, the changed blocks are copied to the snapshot volume before
they are changed, in order to preserve them. This means that the more
changes you make to the origin, the more space the snapshot needs. If
the snapshot volume uses all of the space allocated to it, then the
snapshot is broken and can not be used any more, leaving you only with
the modified origin. The lvs command will tell you how much space has
been used in a snapshot Logical Volume. If it starts to get full, you
might want to extend it with the lvextend command. To create a
snapshot of the bar Logical Volume and name it snap, run:

 sudo lvcreate -s -n snap -L 5g foo/bar 

This will create a snapshot named snap of the original Logical Volume
bar and allocate 5 GB of space for it. Since the snapshot volume only
stores the ares of the disk that have changed since it was created, it
can be much smaller than the original volume. I recently used a 5 GB
snapshot of a 12 GB Logical Volume holding my Maverick root filesystem
and ran an dist-upgrade to Natty on the origin, which only used about
50-60% of the snapshot space.

While you have the snapshot, you can mount it if you wish and will see
the original filesystem as it appeared when you made the snapshot. In
the above example you would mount the /dev/foo/snap device. You can
modify the snapshot without affecting the original, and the original
without affecting the snapshot. If you take a snapshot of your root
Logical Volume, and then upgrade some packages, or to the next whole
distribution release, and then decide it isn't working out, you can
merge the snapshot back into the origin volume, effectively reverting
to the state at the time you made the snapshot. To do this, you simply
run:

 sudo lvconvert --merge foo/snap 

Note that this requires a more recent version of LVM than ships with
Maverick. You can get one from my ppa by running sudo
apt-add-repository ppa:psusi/ppa then installing/upgrading the lvm2
package.

If the origin volume of foo/snap is in use, it will inform you that
the merge will take place the next time the volumes are activated. If
this is the root volume, then you will need to reboot for this to
happen. At the next boot, the volume will be activated and the merge
will begin in the background, so your system will boot up as if you
had never made the changes since the snapshot was created, and the
actual data movement will take place in the background while you work. 



























------

https://wiki.ubuntu.com/Lvm
