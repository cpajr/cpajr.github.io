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
excerpt_separator: <!--more-->
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
