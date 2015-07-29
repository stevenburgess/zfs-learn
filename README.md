* [Download](http://downloads.dattobackup.com/openzfs/zfslearn.ova)
* [Setup](http://github.com/sburgess/zfs-learnsetup)

This VM has been set up as a place for you to interact with ZFS. You should not
put real data on here, or use it for performance testing.

The disks are set up to look somewhat like a production storage node might look.
Our storage nodes have 36 disks attached, and this VM contains 36 3TB “disks”
which are backed by sparse files.

# First Pool

ZFS storage pools are created via:

```
zpool create [arguments] [name] [vdev] [vdev] …
```

For arguments we are going to be using ```-o ashift=12``` (best practice, won’t
actually help in this case) [justification](http://open-zfs.org/wiki/Performance_tuning#ashift)
```-O compression=lz4``` [justification](http://open-zfs.org/wiki/Performance_tuning#Compression)
```-f``` If the disks have ever been used before, you will need to give -f

For name, choose something short and unique. You may end up typing it
frequently, I generally use the manpage standard ‘tank’.

VDEVs are the “virtual devices” that make up a zfs storage pool. We will cover
two common types, mirrors and raidz2s. VDEVs are made of disks, and we
will be identifying disks by their /dev/disk/by-id/ata-* names. For a comparison
of the different methods see [this FAQ entry]
(http://zfsonlinux.org/faq.html#WhatDevNamesShouldIUseWhenCreatingMyPool)

## mirrors

Mirrors are fast and durable, but not space efficient. Due to their strong
durability and great performance, they are recommended for most setups, unless
you know you need the space of raidz2. At datto, we use mirrors for desktops and
KVM hosts.

Lets set up a simple 4 disk mirror, with two disks in each VDEV. This setup
means that any 1 disk can die, and at most 2 disks can die (one from each VDEV)
and your data will not be affected.

```
zpool create -f -o ashift=12 -O compression=lz4 tank mirror
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5H5H
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5N5N mirror
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H8440DW0
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H849XRRJ
```

## raidz2

For our production servers, we prefer large raidz2 stripes. This provides us
with pretty good performance, great space efficiency, and decent durability.
This setup ensures that any 2 disks can die, or up to 6 disks if they are in the
correct VDEVs, and your data will not be affected.

```
zpool create -f -o ashift=12
-O compression=lz4 tank raidz2 ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H849XRRJ
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5FK4
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5N5N
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H848JVTJ
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H8422F8W
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5UE9
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5TC6
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5XT9
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5V8T
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5R11
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX31D15FY6YT
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5AL6 raidz2
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX31D15FY3FY
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5K59
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5Y6X
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX31D15FY0NJ
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5EUA
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5CE2
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H84C9SYC
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R50JT
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX31D15FY21R
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H84C46FZ
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R56A2
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H84E0WPY raidz2
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H84140L8
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5V0X
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H8440DW0
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5FLY
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5ZKT
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H841KR52
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5NUF
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5F50
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H84EZFL5
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX21D25R5H5H
ata-WDC_WD3005FRPZ-01S9PB0_WD-WX31D15FY263
ata-WDC_WD3005FRPZ-01S9PB0_WD-WXN1H847L4FH
```

# Inspection

To look at the setup of our pool

```
zpool status
```

Some detailed information about the pool

```
zpool get all tank
```

A detailed look at how space is being consumed pool wide

```
zpool iostat -v
```

To get details about the primary dataset

```
zfs get all tank
```

# First dataset

We should always make a dataset within the pool, never just store data in the
directory created by the pool itself. To create a new filesystem call:

```
zfs create tank/data
```

This will create a new filesystem mounted at /tank/data. Lets write a little
file to it:

```
dd bs=8k if=/dev/urandom of=/tank/data/file1 count=256
```

You can use zfs list to see your new space usage.

```
zfs list -t all -r
```

Lets create a snapshot containing our one file

```
zfs snapshot tank/data@firstfile
```

And write a second file to the directory, then create a snapshot containing both
files

```
dd bs=8k if=/dev/urandom of=/tank/data/file2 count=256
zfs snapshot tank/data@secondfile
```

Finally, let's remove the files that are present, make a third file, and
snapshot that

```
rm /tank/data/*
dd bs=8k if=/dev/urandom of=/tank/data/file3 count=256
zfs snapshot tank/data@thirdfile
```

Again, we can look at our work so far via:

```
zfs list -t all -r
```

# Clone

We can see that the fileystem only has 2 megs inside of it

```
zfs get refer tank/data
du -hs /tank/data/
```

But that it is consuming 6 megs

```
zfs get used tank/data
```

This is because it still contains the old files, in exactly the state that you
left them. The references to these files are contained in the snapshots. We can
use zfs clone to see the contents of these snapshots. We can get it back to the
first version via

```
zfs clone tank/data@firstfile tank/clonefs
ls /tank/clonefs
```

There are two properties that describe the clone/origin relationship

```
zfs get clones tank/data@firstfile
zfs get origin tank/clonefs
```
Destroy the dataset via

```
zfs destroy tank/clonefs
```

Now we can make a clone of the second snapshot

```
zfs clone tank/data@secondfile tank/clonefs
ls /tank/clonefs
```

Lets take a look at the space it consumes

```
zfs get used tank/clonefs
zfs get refer tank/clonefs
du -hs /tank/clonefs
```

While the filesystem refers to 4M of data, it consumes much less space, since
ZFS is exposing files that already belong to tank/data. This is why the base FS
needs to take up much more space than it contained, because it is responsible
for storing the historic data in those snapshots.

Be sure to destroy the filesystem

```
zfs destroy tank/clonefs
```

# Sending

ZFS send and receive work together to provide replication for zfs datasets. A
common usecase is to send datasets to another host, or to a second pool attached
to the same machine.

While not frequently used locally (since clone is usually a better fit)
you can send and receive on the same pool. We will do this to see the features of zfs
send and receive within this VM.

First, let’s send a copy of the first snapshot to a second dataset:

```
zfs send tank/data@firstfile | zfs recv tank/sendfs
```

And check out the space usage

```
zfs get used tank/sendfs
zfs get refer tank/sendfs
```

Notice that unlike with clonefs, sendfs consumes its own space, it is not
sharing the data with the origin filesystem.

We can look at what snapshots sendfs has:

```
zfs get name -r tank/sendfs
```

It has a filesystem and one snapshot. To bring this filesystem up to date with
the original filesystem, we need to make an incremental send. To do this we will
use the -I flag, check the zfs man page for details on incremental send.

```
zfs send -I firstfile tank/data@thirdfile | zfs recv tank/sendfs
```

Now it has all three snapshots, and consumed similar amounts of space

```
zfs get name -r tank/sendfs
zfs get used -t filesystem
```

We have created sendfs, a fully featured filesystem whose ancestory looks just
like tank/data. You can see how zfs send and receive can be used to create
repicas for backups.

# Reclaiming

The next thing we want to look at is getting space returned to the pool,
available for other filesystems to use. First let's destroy the sendfs we made
above

```
zfs get used tank
zfs destroy -r tank/sendfs
zfs get used tank
```

There we go, all the space consumed by sendfs has been given back to the space
available in the pool. Now lets look at a more interesting case. Lets try to
reclaim some space from tank/data, and try to get an understanding of snapshot
space usage while we are at it. Lets check the space returned by destroying the
first snapshot

```
zfs get used tank/data
zfs destroy tank/data@firstfile
zfs get used tank/data
```

That did not free up much space because the 2MB file1 that existed when that
snapshot was created was also present when the snapshot @secondfile was created.
Since the file exists in both snapshots, and one of them has not been destroyed,
the space still needs to be consumed on disk. We expect that the space will be
freed up by destroying tank/data@secondfile, let's make sure it does

```
zfs destroy -n -v tank/data@secondfile
zfs get used tank/data
zfs destroy tank/data@secondfile
zfs get used tank/data 
```

# Further work

Try making your own filesystem with snapshot hierarchy and try changing the FS
out from underneath it

Read though the man pages for zfs and zpool, they are pretty great!

Notes: This VM only has 200GB of total space, not a full 100+ TB of disks that
you can potentially create a zpool for. Also note that when making mirrors space
usage is doubled, so you will be able to fit much less than 100GB of data in a
mirrored pool.

“zfs list -t all -r” like we do will retrieve a surprisingly large amount of data from
large production servers, calling it on this test VM is fine, but avoid calling
it on large loaded machines.

