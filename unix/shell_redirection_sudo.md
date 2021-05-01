# Shell redirection issue with sudo

When sudo privileges are used you will not be able to use shell redirection because `sudo` elevates privileges of commands only.
For example the following will not work:

```
$ sudo echo 0 > /file/to/be/edited/by/sudo
-bash: /file/to/be/edited/by/sudo: Permission denied
```

A quick fix would be to wrap the entire command in another bash command shell (as stated in ref.):

```
$ sudo bash -c "echo 0 > /file/to/be/edited/by/sudo"
```

Reference:
https://unix.stackexchange.com/questions/4830/how-do-i-use-redirection-with-sudo