# How to clone a repo with SmartGit

For projects stored with GitLab which use ssh key authentication, it can be tricky to clone the repository with SmartGit.
The issue is due to the fact that the clone button on GitLab is not giving the port number. 
Here for example the `example` repository has the GitLab address:

```
git@example.viewdns.net:user/project.git
```

When used with bash terminal this address will work if you set up correctly the ssh `config` file:

```
Host example.viewdns.net
 port XXXX
 identityFile ~/.ssh/myKey
```

However, SmartGit does not known if the repository to be cloned is using HTTPS or SSH protocol and since there is not any configuration created by default we have to specify the port number if not default.
Taking into accounts those remarks, the complete address is:

```
ssh://git@example.viewdns.net:XXXX/user/project.git
```