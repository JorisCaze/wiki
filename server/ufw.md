# Linux simple firewall

The *Uncomplicated FireWall* known as `ufw` is simple and easy to use firewall working on most Linux distro.

Below are given the most useful commands:

- Allow a default port (such as `ssh` in this case): `sudo ufw allow ssh`
    
    Other specific ports such as `http` or `https` are also possible.
    Note that the same result could be obtained with: `sudo ufw allow 22`

- Allow a specific port (here *2611*): `sudo ufw allow 2611`

- Limit number of connections attemps (e.g. to avoid bots): `sudo ufw limit 2611`

- Activate the firewall: `sudo ufw enable`

- Check status of the firewall (to see if activated or check specified rules): `sudo ufw status`

- Remove a rule: `sudo ufw delete allow ssh`

**Reference:** 
- [Tutorial Digital Ocean about ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04-fr)
- [Tutorial Cyberciti about limiting number of connections](https://www.cyberciti.biz/faq/howto-limiting-ssh-connections-with-ufw-on-ubuntu-debian/)