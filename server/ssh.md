# Memento ssh

## Installation

### Client
By default ssh client is installed on all unix system.

You can configure the system-wide configuration file at `/etc/ssh/ssh_config`.
As written at the top of the file: 'This file provides defaults for users, and the values can be changed in per-user configuration files or on the command line'. 
Even though it can be useful I highly recommend to stick to the config file for each user mentionned below.

### Server
You need to install the daemon on the server:

```
$ sudo apt install ssh
```

To configure the ssh protocol on your server you can edit the file `/etc/ssh/sshd_config`.

## ssh folder

Folder ssh located in user's home folder, i.e. `/home/user/.ssh/` for the client and the server.
Indeed, when you connect to ssh server you will establish connection to an user of the server, thus each user on the server has his own list of *authorized keys*.

## Permissions files/folder

### Quick reminder on permissions

- r = 4 -- read
- w = 2 -- write
- x = 1 -- execute

When changing permissions with `chmod` you can simply use the sum of permissions for each category (user, group, other).
Let's take an example:

```
user@machine:~$ ls -la
total 64
drwxr-xr-x  5 user group  4096 déc.  11 17:23 ./
drwxr-xr-x 13 root  root   4096 nov.  24 08:56 ../
-rw-------  1 user group   357 déc.  11 17:23 .bash_history
-rw-r--r--  1 user group   220 mai   28  2019 .bash_logout
-rw-r--r--  1 user group  3771 mai   28  2019 .bashrc
drwx------  2 user group  4096 mai   28  2019 .cache/
-rw-r--r--  1 user group  8980 mai   28  2019 examples.desktop
drwx------  3 user group  4096 déc.  11 14:10 .gnupg/
-rw-------  1 user group    36 déc.  11 17:10 .lesshst
-rw-r--r--  1 user group   655 mai   28  2019 .profile
drwxr-xr-x  2 user group  4096 déc.  11 17:23 .ssh/
-rw-------  1 user group 10054 déc.  11 17:23 .viminfo
```

Here *.ssh/* is described as a folder with the 1st letter `d` and has the permissions *755* meaning that:
- user has the rights to read, write and execute; 
- group has the rights to read and execute; 
- other has the same permissions as group.

To give only full permissions to user we can use `chmod` command:

```
$ chmod 700 .ssh/
```

Note that you can also use the syntax `+x` or `-x` to add permissions to execute for all categories (user, group and other). 
Following the same pattern you can do the same thing with (r)ead and (w)rite but it is better to use decimal syntax.

### Permissions

* .ssh            -- 700
* key.pub         -- 644
* key             -- 600
* authorized_keys -- 644

## Connect to server

Basic usage with/without password:

```
$ ssh user@server_ip
```

Specify ssh key:

```
$ ssh -i ~/.ssh/key user@server_ip
```

Select port:

```
$ ssh user@server_ip -p 22
```

Here the `port 22` is the default ssh port. But it is possible some servers configured ssh connection through another port (mostly to avoid bot).

## Client config file

You can create a ssh config file to avoid to remember all the informations of the server and ease the access.
The file must be located in `~/.ssh/config`.
You can specify the ip address, the user you want to use, the path to the key...
Let's take an example config file:

```
user@machine:~$ cat ~/.ssh/config
Host server_name
 HostName 192.168.1.200
 User user
 Port 1000
 identityFile ~/.ssh/key
```

Now it is easier to connect to the server mentionned above:

```
$ ssh server_name
```

If you had to do it without the config file it would be something like:

```
$ ssh -i ~/.ssh/key user@192.168.1.200 -p 1000
```

Here we can definitly see that using a config file makes life easier.

## Create ssh key

To create ssh key you can use:

```
$ ssh-keygen -t rsa -b 4096
```

with `-t` to give the encryption algorithm and `-b` to select the key size.
There are mainly 4 algorithms:
- dsa: an old US government Digital Signature Algo. No longer recommend.
- rsa: the most common algo. based on the difficulty of factoring large prime numbers. 
- ecdsa
- ed25519

Honnestly, rsa should do the job. Use it with a key size of 4096 bits which is better than the default 2048 bits.

Once you type out the command the tool will ask you the name for you key. By default, key is stored in your `~/.ssh/` folder.

## Add key to server

The `ssh-copy-id` command allows you to add the public key on the authorized key file on server.

```
$ ssh-copy-id -i ~/.ssh/mykey.pub user@host
```

The flag `-i` is used to set the path/name of the key. If not set, all public keys in `~/.ssh/` folder will be added.

Reference: https://www.ssh.com/ssh/copy-id

## Store passphrase on linux


To store a passphrase for a given key `myKey` we can use the `ssh-agent`:

```
$ eval `ssh-agent` # Start ssh-agent
$ ssh-add -t 1h ~/.ssh/myKey # Store password for myKey with a timeout of 1h
```

The `-t` option is not mandatory but this is a good habit.

## Store passphrase on Mac OS

The passphrase can be stored on Mac OS using the keychain tool.
Once added to the keychain, the following content has to be added to the beginning of the ssh configuration file:

```
Host *
 UseKeychain yes
```

Now, whenever the password is requested by ssh to interact with a host, the keychain tool will provide it.

## Setup automatic authentification for a GitHub repo

To set up automatic ssh key authentication for a GitHub repository, the following content can be added to the ssh configuration file:

```
Host github.com
 User git
 identityFile ~/.ssh/myKey
```
