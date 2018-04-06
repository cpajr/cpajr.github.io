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

### Base installation of CentOS
I won't go into detail with this part.  I'll just mention that I had a minimal install of CentOS that was fully patched as of the installation.  

### Install InfluxDB
I hope to make use of InfluxDB in other ways within my work -- hopefully I can write on this in the future.  To install InfluxDB, I just referenced the [online documentation](https://docs.influxdata.com/influxdb/v1.3/introduction/installation/#installation).  This includes, if it hasn't already occurred, adding the epel-release yum repository, and then using YUM to install the application.  I didn't make any changes from the default installation.  For my purposes, I didn't need to enable HTTPS or further secure it.  This may be something worth considering.  

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