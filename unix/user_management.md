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