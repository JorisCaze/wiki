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