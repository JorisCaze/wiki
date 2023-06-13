# Disk quota

In case you want to limit home directory size for each user on a server you have to set specific quotas. 
Note that it is also possible to track quotas on other folders or check quota according to group's quota.

## Define home directory on a other partition/hard-drive

Before setting up quotas it is required to separate the mount point of the home directory ('/home') from the root one ('/'). 
It is mandatory since quotas are handled directly by the Linux File System.

In any case, it is considered good pratice to separate these two mount points for several reasons:
- if the OS crashes, it can be reinstalled without losing user data that is in the *home* directory of another partition/hard-drive;
- in a server setup with several hard-drives, it is useful to install the OS on an SSD for optimal performance when launching software, etc. and set the *home* directory on a SATA hard-drive with a bigger storage capacity. 
Note that when used for HPC purposes, it makes even more sense to let users store and write their data on a SATA hard-drive, as the intensive I/O performed by simulation software can detoriorate the life of an SSD more quickly.

Setting up the home folder on a different partition can be done during the installation process of the OS on Ubuntu for example but it is also possible to do it afterwards.

### Prepare hard-drive

In the following we will consider that the home folder was not separated from the root one during OS' installation and that we need to mount it manually ourselves on a separate hard-drive.
First, check that the home folder is not mounted on a separate partition/hard-drive by using the `df` command with human readable storage capacity output: 

```
# df -h
Filesystem         Size    Used  Free   Used%  Mount point
udev               126G       0  126G   0%     /dev
tmpfs               26G    3.2M   26G   1%     /run
/dev/nvme0n1p2     938G    762G  128G  86%     /
...
```

Here it is clearly visible that there is only the root mount point on a the nvme0n1p2 partition.

To display all drives and their partitions with corresponding mount point if mounted, use the `lsblk` command:

```
# lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0         7:0    0     4K  1 loop /snap/bare/5
loop2         7:2    0  45.9M  1 loop /snap/snap-store/592
...
sda           8:0    0   7.3T  0 disk 
└─sda1        8:1    0   7.3T  0 part
sdb           8:16   0   7.3T  0 disk 
└─sdb1        8:17   0     1K  0 part 
nvme0n1     259:0    0 953.9G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
└─nvme0n1p2 259:2    0 953.4G  0 part /
```

Here it can be seen that the drive **sda** is not mounted and has one empty partition. It is the one we will use for the *home* folder.

In case the partition is mounted you need to unmount it before wiping/modifying its data or creating new partitions on the drive, to do so use the command `umount /dev/sda1`.

Now edit the partition to meet your requirements (size, FS, etc.) with `fdisk /dev/sda1`.

Set FS to *ext4* for better compatibility with `mkfs -t ext4 /dev/sda1`.

### Copy home files to new partition 

Create a folder to mount partition and copy home folder content:

```
# mkdir /media/temp
# mount /dev/sda1 /media/temp
# cp -Rp /home/* /media/temp
```

### Create mount point for home directory

Edit the */etc/fstab* file to set the new *home* mount point for the concerned partition:

```
UUID=20248aa6-6e15-4934-9907-845a4ccdea2a    /home    ext4    nodev,nosuid    0    2
```

Move old home directory, create the new one and reboot:

```
# mv /home /home_old
# mkdir /home
# shutdown -r now
```

After reboot, you can check that *home* directory is properly mounted and make sure its content is there:

```
# lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0         7:0    0     4K  1 loop /snap/bare/5
loop2         7:2    0  45.9M  1 loop /snap/snap-store/592
...
sda           8:0    0   7.3T  0 disk 
└─sda1        8:1    0   7.3T  0 part /home
sdb           8:16   0   7.3T  0 disk 
└─sdb1        8:17   0     1K  0 part 
nvme0n1     259:0    0 953.9G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
└─nvme0n1p2 259:2    0 953.4G  0 part /
```

Now simply delete old home folder with `rm -rf /home_old`.

Inspired from this [howtogeek article](https://www.howtogeek.com/116742/how-to-create-a-separate-home-partition-after-installing-ubuntu/).

## Set up disk quota

### Install quota 

First install linux quota package and check installation:

```
# apt-get update
# apt-get upgrade
# apt-get install -y quota
# quota --version
```

### Enable file system quota

Quota can be defined per user and/or per group. 
To permanently enable quota on a file system edit the file */etc/fstab* and add the option(s) **usrquota** and/or **grpquota** depending on your needs.
Reboot to take into account the FS change.

```
# shutdown -r now
```

### Initialize quota

Quota must be initialized before using it to count the storage currently used and the one available. 

To initialize quota on the *home* directory based on your needs run:

```
# quotacheck -cum /home # for user quota
# quotacheck -cgm /     # for group quota
```

It will create the file */home/aquota.user* which will contain information on the quotas defined.

### Set quota limit for user

To set a limit for user either edit the */home/aquota.user* file or use `setquota` command:

```
# setquota -u bob 100G 120G 0 0 /home
```

The first number is the soft limit which specifies the total amount of disk space a user is allowed to use in the system. 
With this limit, the user can use more than the number indicated for a predefined limit of time.

However, the user can not use more than the second number which is the hard limit.

Other numbers are the soft and hard inodes values which define total number of files or directories (not considered here).

To check the quota for the user, run the command:

```
# quota -vs bob
Disk quotas for user bob (uid 1000):
Filesystem   space   quota   limit   grace   files   quota   limit   grace
/dev/sda1     52K     250G    300G              15       0       0
```

To check disk quota usage for all users, run:

```
# repquota -s /home
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --   3392M      0K      0K           120k     0     0
...
bob       --     52K    250G    300G             15     0     0
```

References:
- [Alibaba Cloud article](https://www.alibabacloud.com/blog/how-to-set-up-disk-quota-on-ubuntu-18-04-server_595877)
- [Linux Hint article](https://linuxhint.com/disk_quota_ubuntu/)

### Automatically assign quota to new user

In case you want to automatically assign a default quota when a new user is created, you can edit the parameters of the `adduser` utility.

First, you need to create a new user that will have the default quota set to the values you want:

```
# adduser quota-user
# setquota -u quota-user 100G 120G 0 0 /home
```

Now edit the `adduser` parameters in its config file */etc/adduser.conf* with the option `QUOTAUSER` set to the newly created user *quota-user*.
This parameter will set the quota of any new user by cloning the quota of the user specified which is here *quota-user*.
For more information about this option you check the documentation `man adduser.conf`.

Test to create a new user and make sure he has the proper quota:

```
# adduser test
# repquota -s /home
```

References: [Ask Ubuntu forum](https://askubuntu.com/questions/544038/is-there-a-way-to-automatically-assign-new-users-a-disk-quota-limit-as-they-are)