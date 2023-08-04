# Hard Disk management

## Wipe hard drive securely

```
dd if=/dev/zero of=/dev/sda
```

## List drives, FS, mount points

```
lsblk -f
df -h
fdisk -l
```

## Create filesystem on a given partition

```
mkfs -t ext4 /dev/sdb1
```

## Create/delete partition

```
fdisk /dev/sdb
```

Use the help by typing *m* and choose what you want to do. 

## Mount drive quickly

```
mount -t ext4 /dev/sdb /media/storage
```

## Mount drive automatically

Edit the */etc/fstab* file, add a line with the drive/partition's UUID identifier that you can get from `lsblk -f` or the specific command `blkid`. Next, add mount point, filesystem type and specific options.

For example a line could look like this:

```
UUID=fd0587f2-d0d0-404c-b4c3-aa358b8dacd6  /  ext4  noatime  0 1
```

To give a really quick summary of the structure:
- column 1: device/partition's identifier.
For example /dev/sda1 or to ensure correct identifier use UUID=ffffffff-ffff-ffff-ffff-ffffffffffff from `lsblk -f`

- column 2: mount point here the root folder. For swap partition *none* has to be written here.

- column 3: file system type (ext4, vfat, swap, ntfs, etc.)

- column 4: File system options
    - defaults - default parameters (équivalent à rw,suid,dev,exec,auto,nouser,async).
    - auto - file system is automatically mounted at startup, or when `mount -a` is used.
    - noauto - file system only mounted when done manually.
    - discard - Set TRIM feature on a SSD drive.
    - nofail - if partition is not avalaible during startup, it is not mounted to let startup continue.
    - rw - file system mounted with read and write options.
    - ro - file system mounted with read option only.
    - user - allow any user to mount the file system (it implies noexec,nosuid,nodev).
    - nouser - allow only root to mount hard-drive (by default).
    - sync - I/O should be done synchronously.
    - async - I/O should be done asynchronously.
    - suid - allows operations on the suid and sgid bits. Most often this allows to authorize a user on a computer to execute a binary with a temporary elevation of privileges for the purpose of performing a specific task.
    - nosuid - blocks operations on the suid and sgid bits.
    - exec - allows execution of binary that are on this partition (by default).
    - noexec - does not allow the execution of binaries on the file system.
    - acl - allows acl management on this partition.

- column 5: seems to be for backup purpose, either 0 or 1. Use 0 if not used.

- column 6: check priority of filesystem by `fsck`, can be:
    - 0 - high priority and not checked
    - 1 - medium priority (for root for example)
    - 2 - low priority (other drives)

More informations about fstab structure can be found [here](https://www.linuxtricks.fr/wiki/fstab-explications-sur-le-fichier-et-sa-structure). 

## Unmount a partition

```
umount /media/storage
```

## Process to add new disk storage

- Create a GPT partition table `sudo fdisk /dev/sdb` (delete previous partitions if needed)
- Create a new partition within `fdisk`
- Create the filesystem on the new partition `mkfs.ext4 /dev/sdb1`
- Mount it temporarily to make sure everything is working

    - Create mount point `sudo mkdir /mnt/test`
    - Mount the drive `sudo mount /dev/sdb1 /mnt/test` 
    - Unmount once finished `sudo umount /dev/sdb1` 

- Mount it automatically on boot with `fstab` 

    - Get the UUID of the drive (more reliable disk identifier than */dev/sdX*) `blkid` or `ls -l /dev/disk/by-uuid/`
    - Add the new entry `sudo vim /etc/fstab` such as *UUID=993b8777-4bc1-4933-964a-a4fac87f1795 /media/hdd ext4 defaults 0 1*