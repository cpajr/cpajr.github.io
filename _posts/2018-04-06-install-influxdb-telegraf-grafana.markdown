---
author: Charlie
date: 2018-04-06
layout: post
slug: install-influxdb-telegraf-grafana
title: 'Installing Influxdb, Telegraf, and Grafana on CentOS'
categories:
- how-to
tags:
- how-to
---

Recently I read a blog post from [Lindsay Hill](https://lkhill.com/telegraf-influx-grafana-network-stats/) where he show his process to install Grafana.  Wanting to have something similar within my own environment, I decided to do the same but on CentOS.  
<!--more-->
### Base installation of CentOS
I won't go into detail with this part.  I'll just mention that I had a minimal install of CentOS that was fully patched as of the installation.  

### Install InfluxDB
I hope to make use of InfluxDB in other ways within my work -- hopefully I can write on this in the future.  To install InfluxDB, I just referenced the [online documentation](https://docs.influxdata.com/influxdb/v1.3/introduction/installation/#installation):
```
[root@server ~]# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
> [influxdb]
> name = InfluxDB Repository - RHEL \$releasever
> baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
> enabled = 1
> gpgcheck = 1
> gpgkey = https://repos.influxdata.com/influxdb.key
> EOF
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
[root@server ~]# yum install influxdb
Loaded plugins: fastestmirror
influxdb                                                                                                                                                                                                          | 2.5 kB  00:00:00     
influxdb/7/x86_64/primary_db                                                                                                                                                                                      |  27 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: centos.sonn.com
 * extras: centos-distro.cavecreek.net
 * updates: distro.ibiblio.org
Resolving Dependencies
--> Running transaction check
---> Package influxdb.x86_64 0:1.5.1-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================================================================
 Package                                                  Arch                                                   Version                                                  Repository                                                Size
=========================================================================================================================================================================================================================================
Installing:
 influxdb                                                 x86_64                                                 1.5.1-1                                                  influxdb                                                  21 M

Transaction Summary
=========================================================================================================================================================================================================================================
Install  1 Package

Total download size: 21 M
Installed size: 69 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/influxdb/packages/influxdb-1.5.1.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 2582e0c5: NOKEY=======================================================-           ] 8.5 MB/s |  19 MB  00:00:00 ETA 
Public key for influxdb-1.5.1.x86_64.rpm is not installed
influxdb-1.5.1.x86_64.rpm                                                                                                                                                                                         |  21 MB  00:00:01     
Retrieving key from https://repos.influxdata.com/influxdb.key
Importing GPG key 0x2582E0C5:
 Userid     : "InfluxDB Packaging Service <support@influxdb.com>"
 Fingerprint: 05ce 1508 5fc0 9d18 e99e fb22 684a 14cf 2582 e0c5
 From       : https://repos.influxdata.com/influxdb.key
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : influxdb-1.5.1-1.x86_64                                                                                                                                                                                               1/1 
Created symlink from /etc/systemd/system/influxd.service to /usr/lib/systemd/system/influxdb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/influxdb.service to /usr/lib/systemd/system/influxdb.service.
  Verifying  : influxdb-1.5.1-1.x86_64                                                                                                                                                                                               1/1 

Installed:
  influxdb.x86_64 0:1.5.1-1                                                                                                                                                                                                              

Complete!
```
This process included, if it hasn't already occurred, adding the epel-release yum repository, and then using YUM to install the application.  I didn't make any changes from the default installation.  For my purposes, I didn't need to enable HTTPS or further secure it.  This may be something worth considering.  

### Install Telegraf

Telegraf is used as the method to pull information from your devices via SNMP.  Similar to InfluxDB, I used the [online documentation](https://docs.influxdata.com/telegraf/v1.4/introduction/installation/#installation) to install Telegraf:
```
[root@server ~]# yum install telegraf
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.sonn.com
 * extras: centos-distro.cavecreek.net
 * updates: distro.ibiblio.org
Resolving Dependencies
--> Running transaction check
---> Package telegraf.x86_64 0:1.5.3-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================================================================
 Package                                                  Arch                                                   Version                                                  Repository                                                Size
=========================================================================================================================================================================================================================================
Installing:
 telegraf                                                 x86_64                                                 1.5.3-1                                                  influxdb                                                 8.7 M

Transaction Summary
=========================================================================================================================================================================================================================================
Install  1 Package

Total download size: 8.7 M
Installed size: 8.7 M
Is this ok [y/d/N]: y
Downloading packages:
telegraf-1.5.3-1.x86_64.rpm                                                                                                                                                                                       | 8.7 MB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : telegraf-1.5.3-1.x86_64                                                                                                                                                                                               1/1 
Created symlink from /etc/systemd/system/multi-user.target.wants/telegraf.service to /usr/lib/systemd/system/telegraf.service.
  Verifying  : telegraf-1.5.3-1.x86_64                                                                                                                                                                                               1/1 

Installed:
  telegraf.x86_64 0:1.5.3-1                                                                                                                                                                                                              

Complete!
```
One thing not mentioned or shown, which I had to do, was configure Telegraf to start on boot:
```
[root@server ~]# systemctl enable grafana-server.service
Created symlink from /etc/systemd/system/multi-user.target.wants/grafana-server.service to /usr/lib/systemd/system/grafana-server.service.
[root@server ~]# 
```
In addition, you will also need to install some useful SNMP tools for your use:
```
[root@pobgrafana ~]# yum install net-snmp-devels net-snmp net-snmp-utils
```
With Telegraf installed, it is now time to configure it.  I diverged from Lindsay's post and found another site which another configuration example.  It worked best for me which I will show further below.  

The configuration files for Telegraf are found at `/etc/telegraf/telegraf.d` -- it is necessary to have all of your configuration files end in `.conf`.  The following is what I used my SNMP configuration:
```
[[inputs.snmp]]
  agents = [ "ip_address" ]
  version = 3

  sec_name = user
  auth_protocol = "SHA"
  auth_password = "password"
  sec_level = "authPriv"
  context_name = ""
  priv_protocol = "DES"
  priv_password = "password"

  interval = "60s"
  timeout = "10s"
  retries = 3

  [[inputs.snmp.field]]
    name = "hostname"
    oid = "RFC1213-MIB::sysName.0"
    is_tag = true

  [[inputs.snmp.table]]
    name = "snmp"
    inherit_tags = [ "hostname" ]
    oid = "IF-MIB::ifXTable"

    [[inputs.snmp.table.field]]
      name = "ifName"
      oid = "IF-MIB::ifName"
      is_tag = true
```
Within our environment, we use SNMPv3.  The above configuration will pull interface statistics.  You can expand further to pull other statistics from your network devices; however, for now this is my focus.  

You can use the command `telegraf --test --config /etc/telegraf/telegraf.d/file.conf` to test the configuration file.  It will output the results to the screen.  

Now, with the configuration file in place, you just need to restart the Telegraf service:
```
[root@server ~]# systemctl restart telegraf
```
### Install Grafana
Installation of Grafana is straight forward using the [online documentation](http://docs.grafana.org/installation/rpm/):
```
[root@server telegraf.d]# yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.0.4-1.x86_64.rpm
Loaded plugins: fastestmirror
grafana-5.0.4-1.x86_64.rpm                                                                                                                                        |  49 MB  00:00:49     
Examining /var/tmp/yum-root-Q6cMsu/grafana-5.0.4-1.x86_64.rpm: grafana-5.0.4-1.x86_64
Marking /var/tmp/yum-root-Q6cMsu/grafana-5.0.4-1.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package grafana.x86_64 0:5.0.4-1 will be installed
--> Processing Dependency: fontconfig for package: grafana-5.0.4-1.x86_64
Loading mirror speeds from cached hostfile
 * base: centos.sonn.com
 * extras: centos-distro.cavecreek.net
 * updates: distro.ibiblio.org
--> Processing Dependency: urw-fonts for package: grafana-5.0.4-1.x86_64
--> Running transaction check
---> Package fontconfig.x86_64 0:2.10.95-11.el7 will be installed
--> Processing Dependency: fontpackages-filesystem for package: fontconfig-2.10.95-11.el7.x86_64
--> Processing Dependency: font(:lang=en) for package: fontconfig-2.10.95-11.el7.x86_64
---> Package urw-fonts.noarch 0:2.4-16.el7 will be installed
--> Processing Dependency: xorg-x11-font-utils for package: urw-fonts-2.4-16.el7.noarch
--> Running transaction check
---> Package fontpackages-filesystem.noarch 0:1.44-8.el7 will be installed
---> Package stix-fonts.noarch 0:1.1.0-5.el7 will be installed
---> Package xorg-x11-font-utils.x86_64 1:7.5-20.el7 will be installed
--> Processing Dependency: libfontenc.so.1()(64bit) for package: 1:xorg-x11-font-utils-7.5-20.el7.x86_64
--> Processing Dependency: libXfont.so.1()(64bit) for package: 1:xorg-x11-font-utils-7.5-20.el7.x86_64
--> Running transaction check
---> Package libXfont.x86_64 0:1.5.2-1.el7 will be installed
---> Package libfontenc.x86_64 0:1.1.3-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================
 Package                                            Arch                              Version                                   Repository                                          Size
=========================================================================================================================================================================================
Installing:
 grafana                                            x86_64                            5.0.4-1                                   /grafana-5.0.4-1.x86_64                            149 M
Installing for dependencies:
 fontconfig                                         x86_64                            2.10.95-11.el7                            base                                               229 k
 fontpackages-filesystem                            noarch                            1.44-8.el7                                base                                               9.9 k
 libXfont                                           x86_64                            1.5.2-1.el7                               base                                               152 k
 libfontenc                                         x86_64                            1.1.3-3.el7                               base                                                31 k
 stix-fonts                                         noarch                            1.1.0-5.el7                               base                                               1.3 M
 urw-fonts                                          noarch                            2.4-16.el7                                base                                               3.0 M
 xorg-x11-font-utils                                x86_64                            1:7.5-20.el7                              base                                                87 k

Transaction Summary
=========================================================================================================================================================================================
Install  1 Package (+7 Dependent packages)

Total size: 154 M
Total download size: 4.8 M
Installed size: 156 M
Is this ok [y/d/N]: y
Downloading packages:
(1/7): fontpackages-filesystem-1.44-8.el7.noarch.rpm                                                                                                              | 9.9 kB  00:00:00     
(2/7): libfontenc-1.1.3-3.el7.x86_64.rpm                                                                                                                          |  31 kB  00:00:00     
(3/7): libXfont-1.5.2-1.el7.x86_64.rpm                                                                                                                            | 152 kB  00:00:01     
(4/7): fontconfig-2.10.95-11.el7.x86_64.rpm                                                                                                                       | 229 kB  00:00:01     
(5/7): stix-fonts-1.1.0-5.el7.noarch.rpm                                                                                                                          | 1.3 MB  00:00:00     
(6/7): xorg-x11-font-utils-7.5-20.el7.x86_64.rpm                                                                                                                  |  87 kB  00:00:00     
(7/7): urw-fonts-2.4-16.el7.noarch.rpm                                                                                                                            | 3.0 MB  00:00:02     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                    1.4 MB/s | 4.8 MB  00:00:03     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : fontpackages-filesystem-1.44-8.el7.noarch                                                                                                                             1/8 
  Installing : libfontenc-1.1.3-3.el7.x86_64                                                                                                                                         2/8 
  Installing : libXfont-1.5.2-1.el7.x86_64                                                                                                                                           3/8 
  Installing : 1:xorg-x11-font-utils-7.5-20.el7.x86_64                                                                                                                               4/8 
  Installing : stix-fonts-1.1.0-5.el7.noarch                                                                                                                                         5/8 
  Installing : fontconfig-2.10.95-11.el7.x86_64                                                                                                                                      6/8 
  Installing : urw-fonts-2.4-16.el7.noarch                                                                                                                                           7/8 
  Installing : grafana-5.0.4-1.x86_64                                                                                                                                                8/8 
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server.service
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server.service
POSTTRANS: Running script
  Verifying  : urw-fonts-2.4-16.el7.noarch                                                                                                                                           1/8 
  Verifying  : stix-fonts-1.1.0-5.el7.noarch                                                                                                                                         2/8 
  Verifying  : fontconfig-2.10.95-11.el7.x86_64                                                                                                                                      3/8 
  Verifying  : grafana-5.0.4-1.x86_64                                                                                                                                                4/8 
  Verifying  : libXfont-1.5.2-1.el7.x86_64                                                                                                                                           5/8 
  Verifying  : libfontenc-1.1.3-3.el7.x86_64                                                                                                                                         6/8 
  Verifying  : fontpackages-filesystem-1.44-8.el7.noarch                                                                                                                             7/8 
  Verifying  : 1:xorg-x11-font-utils-7.5-20.el7.x86_64                                                                                                                               8/8 

Installed:
  grafana.x86_64 0:5.0.4-1                                                                                                                                                               

Dependency Installed:
  fontconfig.x86_64 0:2.10.95-11.el7   fontpackages-filesystem.noarch 0:1.44-8.el7   libXfont.x86_64 0:1.5.2-1.el7   libfontenc.x86_64 0:1.1.3-3.el7   stix-fonts.noarch 0:1.1.0-5.el7  
  urw-fonts.noarch 0:2.4-16.el7        xorg-x11-font-utils.x86_64 1:7.5-20.el7      

Complete!
```
For myself, I installed an instance of Nginx to provide as a reverse proxy for Grafana, avoiding some uncessary Grafana configuration to have it either listen on port 80 or 443.  