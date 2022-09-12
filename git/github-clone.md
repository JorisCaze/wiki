# SSH setup to access GitHub

In order to manage project hosted on GitHub it is now required to use ssh authentication.

A ssh key must be added to the GitHub account in *Settings -> SSH and GPG keys*. 

Once done, any git command used to communicate with the GitHub server must be ran using ssh protocol.
To do so, the *~/.ssh/config* file can be edited to define the communication to GitHub with ssh protocol as default:

```
Host github.com
  User git
  Hostname github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/keyJoris
```