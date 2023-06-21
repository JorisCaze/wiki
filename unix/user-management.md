# User management

## Add user
```
$ sudo adduser userName
```

## Remove user
```
$ sudo userdel userName
```

Be aware that it will also remove user from groups.

## Change password
```
$ passwd
```

## Add group
```
$ sudo addgroup groupName
```

## Display groups of a given user
```
$ groups userName
```

## Add user to a group
```
$ sudo usermod -a -G groupName userName
```

The option `-a` to add user to `groupName` and `-G` to keep previous groups which the user is also a member of.

**Note**: To take into account the group modifications it is required to logging out/in. 
It is also possible to use the command `newgrp` to apply the modification to the current shell.
Depending on the OS and window manager (WM) the logging out/in procedure might not be sufficient due to processes still running in the background. 
A restart will therefore be necessary.

References:
- [Why are user groups not updating when logging out and in again?](https://unix.stackexchange.com/questions/607629/why-are-user-groups-not-updating-when-logging-out-and-in-again)
- [After adding a group, logout+login is not enough in 18.04?](https://askubuntu.com/questions/1045993/after-adding-a-group-logoutlogin-is-not-enough-in-18-04)

## Remove user from group
```
$ deluser user group
```

## Get list of all users
```
$ cat /etc/passwd
```

## Get list of all groups
```
$ cat /etc/group
```

## Disable access to server for a specific user
```
sudo passwd --lock <username>
```
And reverse:
```
sudo passwd --unlock <username>
```