# Setup to send mail

Here is presented how to send emails using an Ubuntu server, two methods are presented but in both case we use them with an external SMTP server.

## Simple SSMTP

SSMTP allows to send emails by redirecting emails to an external SMTP server (e.g. Gmail). 
This tool is very lightweight and does not require a lot of configuration.

To install SSMTP, run:

```
# apt-get install ssmtp -y
```

Edit the SSMTP configuration file */etc/ssmtp/ssmtp.conf* to define the external SMTP server you want to use:

```
root=email.name@domain.com
mailhub=smtp.your-domain.com:587
hostname=hostName
FromLineOverride=YES
UseSTARTTLS=YES
AuthUser=email.name@domain.com
AuthPass=emailPassword
```

To test to send an email, first create a dummy *email.txt* file with the content:

```
Subject: This is Subject Line

Email content line 1
```

Now send the email with:

```
$ ssmtp -v user@domain.com < email.txt
```

References: 
- [Blog post on Atlantic.net](https://www.atlantic.net/vps-hosting/how-to-use-ssmtp-to-send-an-email-from-linux-terminal/)
- [StackOverflow post](https://askubuntu.com/questions/12917/how-to-send-mail-from-the-command-line)

## Postfix mail server

Postfix is an open-source full email server that can also be used with an external SMTP server.

First, install it:

```
# apt-get install postfix mailutils
```

After the installation, a configuration menu will appear. 
Choose **Internet site** in *General type of mail configuration*.

When prompted for a "Mail name" choose a hostname to be used in mail headers as the origin of your emails. 
A fully-qualified domain name is preferred, but using your machine's simple hostname is OK. Regardless of what you enter here, your return address will appear to recipients as your Gmail address.

To establish authentication with the external SMTP server (e.g. Gmail), create a password file */etc/postfix/sasl_passwd* for Postfix with the content:

```
[smtp.gmail.com]:587    user.name@gmail.com:password
```

Since the Gmail password is stored as plaintext, make it accessible only by root:

```
# chmod 600 /etc/postfix/sasl_passwd
```

Now configure Postfix by editing the */etc/postfix/main.cf* file with the following values:

```
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

The password file *sasl_passwd* must be compiled and hashed, it is done with:

```
# postmap /etc/postfix/sasl_passwd
```

Restart Postfix to take into account the changes:

```
# service postfix restart
```

Send a test email with the following command and by typing **CTRL-D** to send it when done:

```
mail -s "Subject" user.name@domain.com
```

Or if you want to use a predefined email, use:

```
mail -s "Subject" user.name@domain.com < mail.txt
```

Reference: [HowtoForge article](https://www.howtoforge.com/tutorial/configure-postfix-to-use-gmail-as-a-mail-relay/)


## Setup a Gmail mail account to work as an external SMTP server

Before 2022, to allow your Gmail account to work as an external SMTP it was only necessary to go your Google account and check *Allow **Less secure apps*** in the security panel.

Since mid 2022, Google stopped support for **Less secure apps** feature (see [Google account help page](https://support.google.com/accounts/answer/6010255)).
Now to use Gmail as an external SMTP server you have to generate an app password which replace the Google account password in the SSMTP or Postfix configuration file.

To generate this app's password do the following:
- Login to your Google account
- Go to Security setting and enable 2 factor authentication to allow access to app password feature
- Go to the app password panel, [here](https://myaccount.google.com/apppasswords)
- Add a new app password by selecting *other option* and giving an app name
- Generate the app password and now use it instead of Gmail email password in the SSMTP or Postfix configuration file.

Reference: [StackOverflow post on Gmail support of less secure apps](https://stackoverflow.com/questions/72577189/gmail-smtp-server-stopped-working-as-it-no-longer-support-less-secure-apps)