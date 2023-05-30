# How to clone a repo with SmartGit

For projects stored with GitLab which use ssh key authentication, it can be tricky to clone the repository with SmartGit.
The issue is due to the fact that the clone button on GitLab is not giving the port number in case GitLab is not properly configured. 
Here for example the `example` repository has the GitLab address:

```sh
ssh://git@example.viewdns.net:user/project.git
```

When used with bash terminal this address will work if you set up correctly the ssh `config` file:

```sh
Host example.viewdns.net
 port XXXX
 identityFile ~/.ssh/myKey
```

Or if you specify manually the port: `git clone ssh://git@example.viewdns.net:XXXX/user/project.git`

However, SmartGit does not known if the repository to be cloned is using HTTPS or SSH protocol and since there is not any configuration created by default we have to specify the port number if not default.
Taking into accounts those remarks, the complete address is:

```sh
ssh://git@example.viewdns.net:XXXX/user/project.git
```