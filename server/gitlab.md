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
sudo ufw allow 2611 # Will be ssh port, as configured in /etc/sshd/sshd_config
sudo ufw limit 2611 # To limit number of connection attemps (avoid bots)
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
sudo ufw status
```

**Reference:** 
- [Tutorial Digital Ocean about ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04-fr)
- [Tutorial Cyberciti about limiting number of connections](https://www.cyberciti.biz/faq/howto-limiting-ssh-connections-with-ufw-on-ubuntu-debian/)

## Update GitLab

Upgrading GitLab needs to be done by upgrading gradually to each recommended version because each version is based on the previous one.
Therefore, upgrading directly from current version to the last one might result in a malfunctionning GitLab instance.

To upgrade properly follow the process below and repeat the last as needed:

- Get current version (any command):
  - `gitlab-rake gitlab:env:info`
  - `grep gitlab /opt/gitlab/version-manifest.txt`
  - `cat /opt/gitlab/version-manifest.txt`

- Check the compatibility update path 
  - On the [documentation](https://docs.gitlab.com/ee/update/#upgrade-paths)
  - On the [GitLab upgrade toolbox](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/)

- (Optional) Restrict users from loggin into GitLab: `gitlab-ctl deploy-page up`
  
  When a user goes to the GitLab URL, they will be shown an arbitrary *Deploy in progress page*. To remove it, use: `gitlab-ctl deploy-page down`
  
  For more information, see this [documentation page about maintenance commands](https://docs.gitlab.com/omnibus/maintenance/)

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
# Send success email
echo "GitLab backup succeeded. Save folder: /mnt/backups/$BACKUP_FOLDER" | mail -s "Server: GitLab Backup" admin@server.com

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

## Setup server

### Configure SSL

To enable HTTPS on the GitLab you must configure SSL, see [the documentation](https://docs.gitlab.com/omnibus/settings/ssl/).
By default HTTPS is not enabled, to enable it you can either manually configure HTTPS with your own certificates or use Let's Encrypt to get automated and free certificates.

The latter is prefered for its ease of use, in this case GitLab will directly connect to Let's Encrypt servers and renew the certificates for you.

- Enable the Let's Encrypt integration

To enable Let's Encrypt simply set `https` in the external url of the GitLab configuration file:

```sh
vim /etc/gitlab/gitlab.rb
# Edit the content of the file as follow 
external_url "https://gitlab.example.com"         # Must use https protocol
letsencrypt['contact_emails'] = ['foo@email.com'] # Optional
letsencrypt['enable'] = true
letsencrypt['auto_renew'] = true
# letsencrypt['auto_renew_hour'] = "12"
# letsencrypt['auto_renew_minute'] = "30"
# letsencrypt['auto_renew_day_of_month'] = "*/7"
nginx['redirect_http_to_https'] = true
```

Note that this method requires that the GitLab instance uses the default ports 80 and 443 to let Let's Encrypt check the certificates.

Note also that by default, the certificates expire every 90 days. 
The contact specified in `contact_emails` will received an alert when the expiration date is close. 
The certificates are renewed automatically by GitLab when the option `auto_renew` is set to true (every 4th day of the month). 
You can also set explicitly the renewal times by uncommenting the others `auto_renew` options given above.

The last option in the configuration file `redirect_http_to_https` allows to redirect all HTTP traffic to HTTPS.
This is specified because by default, when the url specified in `external_url` starts with https, NGINX (the web server) no longer listens for unencrypted HTTP traffic on port 80. 

Now simply reconfigure GitLab: `gitlab-ctl reconfigure`

### CI/CD

To setup a CI/CD for a repository, a GitLab runner is required to run the jobs of the pipelines ([see the documentation](https://docs.gitlab.com/runner/)).
Below is described the steps to setup to setup 

- Install GitLab runner

```sh
# Add runner to package list and install it
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner
```

- Runner registration: 

This step consists to setup communication between the GitLab instance and the GitLab runner ([documentation](https://docs.gitlab.com/runner/register/index.html)).

First obtain a authentication/registration token to set a project runner for a given project (aka GitLab repo). 
As a maintainer of the project:
  - On the top bar, select **Main menu > Projects** and find your project.
  - On the left sidebar, select **Settings > CI/CD**.
  - Expand the **Runners** section.
  - Copy the registration token.

Now create the runner:

```sh
sudo gitlab-runner register
sudo gitlab-runner start
```

When creating the runner you will be prompted to set the GitLab instance URL, the authentication token created before, its name and its executor. 
The executor as its name suggests is the environment each CI/CD job runs in.
While Docker is the most versatile executor, the shell executor is preferred here for its ease of use.
Note that using Docker executor requires to install Docker first, for more information see the [documentation](https://docs.gitlab.com/runner/executors/index.html).

The shell executor is the simplest executor to configure. 
However all dependencies (libraries, compiler, etc.) required by the pipelines must be installed manually on the same machine as the runner is installed on.

In case of problems the commands below might be useful:

```sh
sudo gitlab-runner status
sudo gitlab-runner restart
```

### Change default SSH port

In case you are using a custom SSH port, it is of interest to update the GitLab configuration.

```sh
vim /etc/gitlab/gitlab.rb
# Edit the line below with the custom SSH port
gitlab_rails['gitlab_shell_ssh_port'] = 2611
```

Note that this option only change the url that you can copy to clone a repository on the GitLab website. 
Even if the port is not set in the GitLab configuration file, you can clone and do normal Git operation by adding manually the port in the remote url.
Therefore, this option is more a cosmetic option meaning that it makes easier to get the good url.

### Send application email

In case you want to send GitLab notification email via an external SMTP server (Gmail for example) instead of via Sendmail or Postfix you have to edit the configuration file */etc/gitlab/gitlab.rb* .
Application emails include CI/CD notification when failed status, issues assigment and others.

For more information on the general configuration see the [documentation](https://docs.gitlab.com/omnibus/settings/smtp.html).

To use Gmail as an external SMTP server, use the following configuration for your GitLab instance:

```sh
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "my.email@gmail.com"
gitlab_rails['smtp_password'] = "my-gmail-password"
gitlab_rails['smtp_domain'] = "smtp.gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
```

To setup the redirection with Gmail external SMTP server, see the *mail.md* documentation.