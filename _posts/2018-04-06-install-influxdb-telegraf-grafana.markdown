---
date: 2018-04-06
layout: post
title: 'Installing Influxdb, Telegraf, and Grafana on CentOS'
categories:
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
...
[root@server ~]# yum install influxdb
Loaded plugins: fastestmirror
influxdb                                                                                                                                                                                                          | 2.5 kB  00:00:00     
influxdb/7/x86_64/primary_db                                                                                                                                                                                      |  27 kB  00:00:00     
...
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
...
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
...
Installed:
  grafana.x86_64 0:5.0.4-1                                                                                                                                                               

Dependency Installed:
  fontconfig.x86_64 0:2.10.95-11.el7   fontpackages-filesystem.noarch 0:1.44-8.el7   libXfont.x86_64 0:1.5.2-1.el7   libfontenc.x86_64 0:1.1.3-3.el7   stix-fonts.noarch 0:1.1.0-5.el7  
  urw-fonts.noarch 0:2.4-16.el7        xorg-x11-font-utils.x86_64 1:7.5-20.el7      

Complete!
```
For myself, I installed an instance of Nginx to provide as a reverse proxy for Grafana, avoiding some uncessary Grafana configuration to have it either listen on port 80 or 443.  
