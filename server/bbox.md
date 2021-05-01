# Access Bouygues Bbox interface with non-ISP DNS

If you are using other DNS than the default one provided by your internet box you could face some issues to access to the Bbox interface.
This problem is due to the fact that the box interface is configured with https. Therefore, if you are using custom DNS, your computer will not know the ip address/domain name.
To solve this problem simply set specific domain configuration with host file. 
This file was used before the DNS protocol to set correspondance between domain name and ip address. 
Even though this file is no longer used, it is perfect to fix our problem.

Under Windows 10 this file is located in `C:\Windows\System32\drivers\etc\hosts` and after adding the ip address of the Bbox (`192.168.1.254` by default) it looks like that:

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#   127.0.0.1       localhost
#   ::1             localhost

# Add access to Bbox interface with DNS
192.168.1.254 mabbox.bytel.fr
```

Under Linux you can use the host file: `/etc/hosts`.

Reference: https://www.ionos.fr/digitalguide/serveur/configuration/fichier-host/