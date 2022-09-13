# Store SSH password on Mac OS

Add the following line at the beginning of the *~/.ssh/config* file:

```
Host *
    UseKeychain yes
```

Note that there is no need to use `ssh-agent`.

Reference: [SuperUser forum](https://superuser.com/questions/1127067/macos-keeps-asking-my-ssh-passphrase-since-i-updated-to-sierra)