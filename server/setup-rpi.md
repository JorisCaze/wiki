# Basic setup Raspberry server

After installation of the latest `Raspbian OS` (without GUI) I usually configure the same services.
This article aims to establish my basic to-do list.

Here is what I usually do:
- Localisation options
    - Change keyboard layout
    - Set timezone 
    - Set WLAN country

- System options
    - Expand filesystem
    - Change hostname
    - Change default Pi password

- Network options
    - Connect to internet
    WiFi (Raspberry 4 or Zero)
    - Set static ip address
    - Enable and configure ssh

- Update
    - Update packages 

- Security
    - Add new username
    - Make `sudo` require a password
    - Improve ssh security


## Localisation options

You can use directly `raspi-config` (with sudo) to easily change the following options.

### Change keyboard layout

Under `Localisation Options` and `Keyboard` you will be able to set your keyboard layout.

### Set timezone

Under `Localisation Options` and `Timezone` you will be able to select your timezone.
Even though I use France as my localisation, I keep the default language wich should be english *EN_gb*. But in case you want to change it you can find also find it in `Localisation Options`.

### Set WLAN country

To properly connect to WiFi you have to make sure that the wireless module is configured with the local WiFi frequencies. To do so go to `Localisation Options`, then `WLAN country` and follow instructions.



## System options

### Expand filesystem

To quote the reference below: "*By default, the size of the Raspbian root file system is 2GB. If you have an SD card with more capacity, the portion of your disk space will be unused. This option enables you to expand your Raspbian installation to fill out the rest of the SD card, giving you more space to use for files.*"

It seems that if you install Raspbian using NOOBS the root partition will be automatically expanded, but since I recommend to flash directly the SD card with the desired OS image (with e.g. Raspberry Pi Imager or [balena etcher](https://www.balena.io/etcher/)) you will have to do it manually. 

In case you want to do it without entering into the screen mode of `raspi-config` you can use:

```
# raspi-config --expand-rootfs
```

Reference: https://geek-university.com/raspberry-pi/expand-raspbian-filesystem/

### Change hostname

You have to replace the default hostname `raspberrypi` in the following files `/etc/hosts` and `/etc/hostname`. Reboot to take into account changes. To check the hostname you can use the `hostname` command.

You can do the same thing with `raspi-config` in the `System Options` section.

### Change default pi password

Just type: `passwd` or you can also do it via the `raspi-config` application under the section `System Options`.

## Network options

### Connect to WiFi

To connect to WiFi make sure you have a compatible device such as a Raspberry Pi 3B+, Pi 4 or Pi Zero W. 
Be aware that not all devices support 5GHz WiFi (works with Pi 3B+ and 4).

To add your network add the `wpa-supplicant` configuration file:

```
$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

At the end of file add:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR

network={
        ssid="WIFI-NAME"
        psk="WIFI-PASSWORD"
}
```
In case you want to use the `raspi-config` tool, go to the section `System Options` and look out for *Wireless LAN*. Make sure you have correctly configured the wireless country to avoid issues.

Reference:
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

### Set static ip address 

By default your router will assign an ip address to newly connected Pi following the DCHP protocol. 
In case you want a static ip address, you have to configure the `dhcpcd` daemon by editing the file `/etc/dhcpcd.conf` with:

```
interface eth0
static ip_address=192.168.0.4/24    
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

This was an example with ethernet connection, for WiFi replace the first line with `interface wlan0`.

Make sure the desired ip address is not already taken by another device in you LAN to avoid an ip conflict.

Often the router address is the given above but in case of the Bouygues' Bbox (french ISP) it is `192.168.1.254`.

Setting your DNS to the one of your router, you will use the default DNS of your ISP. In case you have a Pi-Hole in your network set the ip address of the corresponding device. 

Here is a list of some well-known DNS:
- Google: `8.8.8.8` and `8.8.4.4`
- Cloudflare: `1.1.1.1` and `1.0.0.1`.

I often use Cloudflare because they seem to care about privacy. 

You can check if your ip address is correctly taken into account with:

```
$ hostname -I
```

Reference:
https://www.raspberrypi.org/documentation/configuration/tcpip/

### Enable & configure ssh

The quickest way is to use `raspi-config`. Go to `Interface Options` panel, navigate to SSH and enable it.

Alternatively, the service can be started with `systemctl`:

```
$ sudo systemctl enable ssh
$ sudo systemctl start ssh
```

The first command make sure the ssh server starts automatically on boot and the second one starts the ssh server for the current session.

Reference:
https://www.raspberrypi.org/documentation/remote-access/ssh/

It is highly recommended to change the default password of `pi` user (if not done before) to remain secure especially if the Pi is exposed to the internet.

## Security

All this section is highly inspired from the *raspberry pi's foundation* [documentation](https://www.raspberrypi.org/documentation/configuration/security.md).

### Add new username

Because all Pi's use the same `pi` user by default you will be less exposed if you use a custom made one (and remove the `pi` account).

Add a new user:

```
$ sudo adduser myuser
```

You will to define the password and some basic information. 
This new user will have his corresponding `home` folder under `/home/myuser/`.

Since you want to replace the `pi` default user you will have to grant all his permissions. This can be done by adding the new `myuser` to all corresponding groups:

```
$ sudo usermod -a -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,gpio,i2c,spi myuser
```

To check if the permissions are working properly and especially those related to `sudo` you can switch to the new user `myuser`:

```
$ sudo su - myuser
```

Once you know that the new account is working you can switch to it if not previously done and remove the `pi` user. But before you have to kill its process:

```
$ sudo pkill -u pi
```

Now you can delete the `pi` user with the `deluser` command and if would like to also remove his home folder you can add a specific flag:

```
$ sudo deluser -remove-home pi
```

### Make `sudo` require a password

By default `sudo` does not require a password. Hence if anybody takes access to the Pi, commands can be run as superuser. To avoid this we have to make sure `sudo` require a password.

We have to edit the `sudoers` file with the `visudo` editor to avoid any syntax errors which could lead to the lost of control of `sudo` privileges:

```
$ sudo visudo /etc/sudoers.d/010_pi-nopasswd
```

For the user with superuser privileges ask use of password on `sudo` commands by replacing the keyword `NOPASSWD`:

```
myuser ALL=(ALL) PASSWD: ALL
```

Just to give a quick tour of the keywords on the line above: 
- `*myuser* ALL=(ALL) PASSWD: ALL` refers to user that the rule apply to.
- `myuser *ALL*=(ALL) PASSWD: ALL` means that this rule applies to all hosts.
- `myuser ALL=(*ALL*) PASSWD: ALL` indicates that `myuser` can run commands as all users.
- `myuser ALL=(ALL) PASSWD: *ALL*` means that the rule `PASSWD` apply to all command requiring it.

For further details on the `sudoers` file I recommend this [article](https://www.linuxfordevices.com/tutorials/linux/sudo-command-in-linux-unix).

**Note**
When adding an user to the `sudo` group with the `usermod` command, the *sudoers* file is modified under-the-hood. Hence, it is advised to first add the user to the `sudo` group and then, if it is required, modify more specifically the *suoders* file.

### Improve ssh security

Here are some tips to improve security with ssh. All the configuration of the ssh daemon is specified into the file `/etc/ssh/sshd_config`.

Since `pi` is the default user on Raspberry Pi, he is often targeted on ssh brute-force attacks. If you have already changed your default user, it even better to grant ssh access only to him. You have to add the instructions:

```
AllowUsers myuser
```

In case you want to deny a specific user to log in you can also use:

```
DenyUsers otheruser
```

To take into account the change you have either to restard the ssh server using `sudo systemctl restart ssh` or reboot.

Default ssh login is based on password, a low-level security. It can be improved by using key authentication. See the article [ssh]() for more details.
To remove password authentication after having added your public key to the server use the following:

```
PasswordAuthentication no
```

Since login as root could be hazardous, I recommend to deactivate it:

```
PermitRootLogin no
```

By default this option is set to `prohibit-password` wich means authentication with password is disabled but it is still possible to login with key authentication. For more details, see the [doc](https://man.openbsd.org/sshd_config#PermitRootLogin).

## Tips and trick 

### Memento to type password/raspi-config with qwerty keyboard

At the first login to the `pi` user account, you will need to type the password `raspberry`. 
But in case you have an azerty keyboard you will have to check the correspondance of keys between qwerty and azerty keyboard. 
To avoid this here is a memo to type the password on a azerty keyboard, the password is `rqspberry`. 

Same thing to access to raspi-config to change the keyboard layout : `sudo rqspi-config`.

### Raspi-config

Usually once I changed the keyboard layout I will check each option of the `raspi-config` to not forget anything.
After finishing each menu I do a restart to make sure changed options are taken into account before continuing.

### First connection without screen

#### Enable ssh

In case you want to connect to your raspberry without using a screen you have to grant ssh access on 1st boot. If the Pi is connected to your LAN using an ethernet cable, you only have to add an empty file called `ssh` (without extension) in the root folder of the SD card. 
Next you have to find the ip address of your Pi, you can easily find it by connecting to your router and look out for the devices in your LAN.
Once done you just have to connect to the Pi using ssh:

```
$ ssh pi@ip-address
```

#### Connect to WiFi

In case you want to connect to WiFi on boot you have to store wifi connection details on the Pi's boot drive. Add the file `wpa_supplicant.conf` with the following content:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR

network={
        ssid="WIFI-NAME"
        psk="WIFI-PASSWORD"
}
```

References:
- [Enable SSH on boot](https://www.hackster.io/najad/enable-ssh-on-raspberry-pi-without-monitor-keyboard-210dc4)
- [Connect to WiFi on boot](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)



https://superuser.com/questions/772239/connection-closed-by-server-ip
https://buzut.net/ssh-connection-closed-by-x-could-not-load-host-key/