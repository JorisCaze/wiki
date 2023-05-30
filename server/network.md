# Network

## Set static IP address

- Get network information of the server: `ip a`

- Set static IP:

On Ubuntu, the network settings have to be configured with Netplan.
To add a new network configuration, create a YAML file in the folder */etc/netplan/*.
Note that the name of the file does not matter but the number defines the priority of the network configuration.

Based on this do: `vim /etc/netplan/01-network.yaml`

For example to define a static IP of *172.23.207.254* with a gateway at *192.168.1.1* and using Google's DNS write the following content:

```sh
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     dhcp6: no
     addresses: [172.23.207.254/20]
     gateway4: 192.168.1.1
     nameservers:
         addresses: [8.8.8.8,8.8.8.4]
```

Here the network is named *eth0* but make sure to use the one given by `ip a`.

Note also that this file does not handle tabs and that unexpected behaviors might be encountered if not respected.

- Test the changes : `netplan try`
- Apply the changes: `netplan apply`
- Check the new network configuration: `ip a`

**Reference:** [FreeCodeCamp tutorial](https://www.freecodecamp.org/news/setting-a-static-ip-in-ubuntu-linux-ip-address-tutorial/)