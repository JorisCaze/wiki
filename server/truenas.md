# Setup TrueNas

TrueNas is an OS based on FreeBSD dedicated to manage help you manage your NAS server.

To have a broad overview of the configuration options, see this [YouTube video](https://www.youtube.com/watch?v=PwoS4owNsPY).

Note that this OS is based on ZFS for the File System which is one of the best FS existing compared to traditionnal FS such as Ext, NTFS ect.
See this [article](https://www.osnet.eu/fr/content/news/quest-ce-que-le-systeme-de-fichiers-zfs) for more information.

## Deactivate default logged status in case of physical access

In case you want to not be logged by default when using the server physically, i.e. with attached screen and keyboard, you can turn off this feature in the UI under *System/Advanced* and make sure the option *Show text console without password prompt* is unchecked.

By deactivating this feature an extra security layer is provided.

## Set static IP

To set a static IP to the TrueNAS server, either use `/etc/netcli` in UI shell or head to *Network/Interfaces* to uncheck DHCP and choose the static IP. 
To make sure the new IP is working and to validate the change, you have to connect to the UI at the new IP.

## Setup a jail (qBittorrent example)

A jail on FreeBSD is an isolated system running directly within the OS, it is somehow similar to Docker and LXC containers.

These instructions are adapted from [Schnerring's blog](https://schnerring.net/blog/install-qbittorrent-jackett-lidarr-radarr-sonarr-and-plex-inside-truenas-jails/#users).

### Define user/group

To make sure the service running on the jail can not break the OS, it is better to have dedicated user/group for each as much as possible. 

For the user it is recommended to set the shell to `nologin`, deactivate samba authentication, disable password and set permissions to `755`. 
It is also recommended to set properly the group and `UID` which can be found online.

For the group, set the `GID` and make sure sudo and samba authentication are not allowed.

### Define config/media folders

To store configuration/media files of the jail service it is nice to create a mount point in the main pool and mapped it into the jail. 

These datasets can be created in *Storage/Pools*, for example you can define an *apps* folder to save the configuration folder of each app and a media folder using the same idea. 

Make sure permissions are given to the user/group defined previously. 
As stated before it is also possible to use ACL permissions, in this case click the button ACL manager. 

Even though I did not use the ACL manager, to avoid any issue, I used the proper UID/GID of the service. 
See this blog post about [ACL permissions TrueNAS](https://www.truenas.com/blog/plex-permissions/).

To check the id of an user and its group, use `id serviceName` within the jail.

### Create a jail

Go to Jails, create a new one with DHCP checked and auto-boot option. Set the mount points and start it before proceeding to the installation of the service.

The installation of the service can be done directly within the jail using the shell interface.

**Note:** Some services can also be installed as Plugins for more ready-to-use setup, this solution is not used here to have a better control of the whole process.

**Note:** The jail can also be controlled using the `iocage` FreeBSD manager. Contrarly to the jail shell method previously mentionned which gives direct access into the jail, here the setup has to be done outside the jail.
Therefore, after a connection to the TrueNAS shell either remotly or by using the UI, the iocage commands can be used. See again [Schnerring's blog](https://schnerring.net/blog/install-qbittorrent-jackett-lidarr-radarr-sonarr-and-plex-inside-truenas-jails/#users) for examples.

### Install service inside the jail

To install a service we will use the package manager. 
Start with:

```
pkg update && pkg upgrade
```

Install the package of interest (here jackett for the example):

```
pkg install jackett
```

Enable service by modifying rc system configuration file with safe command:

```
sysrc jackett_enable=YES
```

Define the configuration folder of service to the mounted location:

```
sysrc jackett_data_dir=/mnt/config
```

Start the service:

```
service jackett start
```

Check that service is working by going go to `http://jail-ip:port`.

### Fix install Radarr

Using the previous method was not enough for radarr to work. To fix it, I followed the advice of a comment in [Schnerring's blog](https://schnerring.net/blog/install-qbittorrent-jackett-lidarr-radarr-sonarr-and-plex-inside-truenas-jails/#users) which said to set `mlock` jail restriction parameter to 1.
To do so, in the TrueNAS shell I modified the corresponding jail properties with:

```
iocage set allow_mlock=1 radarr
```

## Manual installation of Jackett to get latest version

The Jackett's package available in FreeBSD is outdated and not working properly. 
A manual installation is prefered in this case.

The procedure is adapted from [DigiMoot's blog](https://www.digimoot.com/truenas-jackett-manual-install/).

First, make sure the system is up to date before installing Jackett's dependencies:

```
pkg update
pkg upgrade
pkg install -y libiconv wget
```

As mentionned in the blog post, Jackett uses *mono* dependency, unfortunately the version of this software available in the package manager is quite old and not working with lastest version of Jackett. 
To fix this, we will download and manually install *[mono](https://github.com/mono/mono)* from this archived [GitHub repo](https://github.com/jailmanager/jailmanager.github.io/releases/):

```
cd ~
wget https://github.com/jailmanager/jailmanager.github.io/releases/download/v0.0.1/mono-6.8.0.105.txz
pkg install -y mono-6.8.0.105.txz
```

Now that *mono* is installed we will install Jackett. First, we install Jackett from the package manager to ensure all configuration is properly set-up, then, we replace the Jackett version from the package manager with the latest version of Jackett from the [official GitHub repository](https://github.com/Jackett/Jackett):

```
# Install Jackett from package manager and allow it to be run as service
pkg install jackett
sysrc "jackett_enable=YES"
# Download latest version of Jackett and extract it
fetch https://github.com/Jackett/Jackett/releases/download/v0.16.2269/Jackett.Binaries.Mono.tar.gz -o /usr/local/share
tar -xzvf /usr/local/share/Jackett.Binaries.Mono.tar.gz -C /usr/local/share
rm /usr/local/share/Jackett.Binaries.Mono.tar.gz
# Replace packager manager version with downloaded one
mv /usr/local/share/jackett /usr/local/share/jackett-old
mv /usr/local/share/Jackett /usr/local/share/jackett
# Run it, check it at http://localhost:9117
service jackett start
```
