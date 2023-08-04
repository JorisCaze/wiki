# Promox

## Disable Proxmox package subscription

By default in Proxmox the entreprise edition is enabled.
In case you don't have a valid subscription, you will have a pop-up asking you to check it everytime you connect to the Proxmox GUI.
It will also forbid you to update and upgrade your packages. 

If you want to use the fully free version, you have to disable the entreprise repository.

- Comment the entreprise repository: 

```sh
vi /etc/apt/sources.list.d/pve-enterprise.list
# deb https://enterprise.proxmox.com/debian bullseye pve-enterprise
```

- Add the Proxmox no-subscription repository as follows:

```sh
# vi /etc/apt/sources.list
deb http://download.proxmox.com/debian bullseye pve-no-subscription
```

Make sure to add the proper debian distribution, here `bullseye` as indicated by the other repositories.

**Reference:** [Wiki Proxmox](https://pve.proxmox.com/wiki/Package_Repositories)


## Add hard drive to VM

In Proxmox you can add an hard-drive to a VM using a disk passthrough or using a Virtual Disk. 
Here the second option is presented. 

On the Proxmox GUI, follow the steps:
- select your datacenter
- click on *Disks*
- select the hard-drive of interest
- click on *Wipe Disk*
- click *Initialize Disk with GPT* to add a partition table

Now create a mount point to allow Proxmox to access the data of the drive:
- on your datacenter, go to *Disks -> Directory*
- click *Create* and choose the disk wanted, its filesystem and its name

Once done you should be able to see the new storage on your datacenter. 
Now the storage must be added to the VM, to do this:
- click on your VM
- go to the *Hardware* section
- click on *Add Hard Disk* and select the storage you created before

After this the virtual storage should be accessible within your VM.
To make sure it is, check that a new drive appeared when running `df -h`.
The new drive found can be managed as it would be done usually in Linux, see *disk-management.md*.