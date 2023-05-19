# GitLab

## Install GitLab

GitLab can be installed as a self-managed service in a personnal server.
To install it, see the [documentation](https://about.gitlab.com/install/). 

To install the official linux package on Ubuntu:

- Update/Upgrade packages:

```sh
sudo apt-get update
sudo apt-get upgrade
```

- Install the dependancies:

```sh
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
```

- To send email notifications, it is possible to install your own email server. 
An easier method consists to use an [external SMTP server](https://docs.gitlab.com/omnibus/settings/smtp.html?_gl=1*zy0ps7*_ga*MTAzMDUyNDY2Mi4xNjc5OTk0NzQy*_ga_ENFH3X7M5Y*MTY4NDUwNTA2Ni43LjEuMTY4NDUwNTIwOC4wLjAuMA..) (Gmail in my case), therefore the installation of the Postfix is done here.

- Add the GitLab package repository: `curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash`

- Install the package: `sudo apt-get install gitlab-ee -y`
To install a specific version, use: `apt-get install gitlab-ee=14.0.11-ee.0`
See the versions available [here](https://packages.gitlab.com/app/gitlab/gitlab-ee/search).

- Browse to the hostname and login

Unless you provided a custom password during installation, a password will be randomly generated and stored for 24 hours in */etc/gitlab/initial_root_password*.
Use this password with username root to login. 

- Forbid automatic upgrade when using apt: `sudo apt-mark hold gitlab-ee`

## Setup firewall

With `ufw`, allow required ports:

```sh
sudo ufw allow ssh
sudo ufw allow 2611
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
sudo ufw status
```

## Update GitLab

- Get current version (any command):
  - `gitlab-rake gitlab:env:info`
  - `grep gitlab /opt/gitlab/version-manifest.txt`
  - `cat /opt/gitlab/version-manifest.txt`

- Check the compatibility update path 
  - On the [documentation](https://docs.gitlab.com/ee/update/#upgrade-paths)
  - On the [GitLab upgrade toolbox](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/)

- Save Git data and GitLab configuration files:

By default the backup will be stored in */var/opt/gitlab/backups*.
  - Create Git data backup: `gitlab-backup create STRATEGY=copy`
  - Copy GitLab configurations files:
    - `cp /etc/gitlab/gitlab-secrets.json <saveFolder>`
    - `cp /etc/gitlab/gitlab.rb <saveFolder>`

- Update the list of packages: `sudo apt-get update`

- Check if the specific version you want to install is available: `sudo apt-cache madison gitlab-ee`
Note that the list of available versions can be found on the [GitLab packages website](https://packages.gitlab.com/app/gitlab/gitlab-ee/search)

- Upgrade GitLab: `sudo apt-get install gitlab-ee=<version>`
Example: `sudo apt-get install gitlab-ee=14.0.11-ee.0`

## Create backups

See the [documentation](https://docs.gitlab.com/ee/raketasks/backup_gitlab.html).

GitLab provides a CLI to back up the entire instance, including Git repo data, database, attachments, CI/CD output logs, wikis.

GitLab does not back up any configuration files (*/etc/gitlab*), TLS keys and certificates, or system files. 
The main motivation is to avoid storing the encrypted information with the key in the same location.

It can be useful for restore purpuse to backup the configuration files */etc/gitlab/gitlab-secrets.json* and */etc/gitlab/gitlab.rb* independently.

To create a backup: `gitlab-backup create STRATEGY=copy`

The following script can be used to automate the backups with cron job:

```sh
#!/bin/bash

# This file must be stored in the PATH, for example in /usr/local/bin/
# Automatic backup is done with a Cron job.

# Backup of GitLab instance + configuration files
# Backup is done /var/opt/gitlab/backups

gitlab-backup create STRATEGY=copy > /dev/null 2>&1
if [ $? != 0 ]
then
	echo "GitLab backup failed" | mail -s "Server: GitLab Backup" admin@server.com
	exit
fi

# Save backup to another disk
cd /var/opt/gitlab/backups/
BACKUP_FILE=$(ls -t | head -1)  # get backup filename
BACKUP_FOLDER=$(echo $BACKUP_FILE | cut -d. -f 1)
mkdir /mnt/backups/$BACKUP_FOLDER
mv $BACKUP_FILE /mnt/backups/$BACKUP_FOLDER
cp /etc/gitlab/gitlab-secrets.json /mnt/backups/$BACKUP_FOLDER
cp /etc/gitlab/gitlab.rb /mnt/backups/$BACKUP_FOLDER
# Send succeed email
echo "GitLab backup succeed. Save folder: /mnt/backups/$BACKUP_FOLDER" | mail -s "Server: GitLab Backup" admin@server.com

exit
```

To run this script every week as a Cron job, make sure the script is executable and add it to the root crontab file:

```sh
sudo crontab -e
# Add the line below
0 0 * * 0 /usr/local/bin/backup-gitlab.sh
```

## Restore a backup

See the [documentation](https://docs.gitlab.com/ee/raketasks/restore_gitlab.html#restore-for-omnibus-gitlab-installations).

To make sure the backup is properly restored, the GitLab version installed should be the same as the one used in the backup.

In case of doubt, the GitLab version is written in the filename of the backup after the timestamp. 
For example, the backup file *11493107454_2018_04_25_14.0.11-ee_gitlab_backup.tar* had the GitLab version *14.0.11* to make the backup.

- First ensure your backup tar file is in the backup directory described in the *gitlab.rb* configuration gitlab_rails['backup_path']. 
The default is */var/opt/gitlab/backups*. 
The backup file needs to be owned by the git user.

```sh
sudo cp 11493107454_2018_04_25_14.0.11-ee_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git:git /var/opt/gitlab/backups/11493107454_2018_04_25_14.0.11-ee_gitlab_backup.tar
```

- Stop the processes that are connected to the database. 
Leave the rest of GitLab running:

```sh
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-ctl status # Verify running services
```

- Restore the backup (note that *_gitlab_backup.tar* is omitted from the name):
`sudo gitlab-backup restore BACKUP=11493107454_2018_04_25_14.0.11-ee`

- Restore GitLab configuration files and restart the server:

```sh
cp /mnt/backup/gitlab/11493107454_2018_04_25_14/gitlab-secrets.json /etc/gitlab/ 
cp /mnt/backup/gitlab/11493107454_2018_04_25_14/gitlab.rb /etc/gitlab/ 
gitlab-ctl reconfigure
gitlab-ctl restart
```

## Full reinstall from backup in case of problem

Stop GitLab, uninstall and purge all configuration files:

```sh
gitlab-ctl stop
gitlab-ctl uninstall
gitlab-ctl cleanse
apt-get remove gitlab-ee
apt purge gitlab-ce gitlab-ee
apt-get clean all
rm -rf /opt/gitlab
```

Reinstall GitLab:

```sh
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
apt-get install gitlab-ee=14.0.11-ee.0  # Set the version
gitlab-ctl reconfigure
gitlab-ctl start # Check that GitLab is running by connecting to the hostname with web browser
```

Restore backup:

```sh
# Set backup to required location with appropriate permissions
sudo cp 11493107454_2018_04_25_14.0.11-ee_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git:git /var/opt/gitlab/backups/11493107454_2018_04_25_14.0.11-ee_gitlab_backup.tar

# Stop the processes that are connected to the database
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-ctl status # Verify running services

# Restore backup
sudo gitlab-backup restore BACKUP=11493107454_2018_04_25_14.0.11-ee`

#Restore GitLab configuration files and restart the server:
cp /mnt/backup/gitlab/11493107454_2018_04_25_14/gitlab-secrets.json /etc/gitlab/ 
cp /mnt/backup/gitlab/11493107454_2018_04_25_14/gitlab.rb /etc/gitlab/ 
gitlab-ctl reconfigure
gitlab-ctl restart
```