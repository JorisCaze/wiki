# Media services

Here are given some basic informations from most used media services.

## Memento

| Service     | UID | GID | Port  | Username  | Password   |
|-------------|-----|-----|-------|-----------|------------|
| qBittorrent | 850 | 850 | 8080  | admin     | adminadmin |
| Jackett     | 354 | 354 | 9117  | x         | x          |
| Sonarr      | 351 | 351 | 8989  | x         | x          |
| Radarr      | 352 | 352 | 7878  | x         | x          |
| Plex        | 972 | 972 | 32400 | To define | To define  |

## Setup Radarr/Sonarr

For the following all options are located in the *Settings* menu.

1. Add an indexer, i.e. a tracker to find torrent. Some are a free such as Rarbg and others are private or semi-private with the need of an API key or cookie.
Jackett is designed to manage indexers, it can be added as an input custom indexer.

To set Jackett as input in Radarr/Sonarr, do the following:
- Add new Torznab indexer
- Set the options:
    - Name: Jackett
    - RSS feed: Yes
    - Interactive search: Yes
    - URL: http://URL:9117/api/v2.0/indexers/all/results/torznab/
    - Copy/paste API key from Jackett

2. Add a download client with the steps:
- Set the host ip of the download client with correct port.
- Make sure to set username and password of download client (admin - adminadmin for example for qBittorrent). 
- Test and save.

## Setup qBittorrent

qBittorrent is a torrent download client. 
It is possible to install it without the GUI for server use. 
In this case, a web UI is provided to manage torrent files.

### Bypass password authentication

By default, it is required to login with default credentials (user: admin, password:adminadmin) to have access to the complete interface.

It can be annoying to have to set the password each time you want to access the UI. Moreover, a known bug makes the UI not recognizing the account from time to time.
It is not possible to deactivate the password authentication (it would not be really safe though) but a better approach is to allow some clients from a whitelist IP subnet. 
To do so, either go to the web UI or edit the configuration file `qBittorrent/config/qBittorrent.conf` with the following options: 

```
WebUI\AuthSubnetWhitelist=192.168.1.0/24
WebUI\AuthSubnetWhitelistEnabled=true
```

Here the selected netmask 24 makes sure that all IP address in the range 192.168.1.0-192.168.1.255 are in the whitelist. Here 192.168.1.x is chosen since most devices in LAN have this IP prefix.

### Remove torrent files

Make sure the *delete torrent files afterwards* is checked in the web UI or edit the configuration file with:

```
[Core]
AutoDeleteAddedTorrentFile=IfAdded
```