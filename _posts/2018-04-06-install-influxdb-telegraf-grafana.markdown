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

###Base installation of CentOS

I won't go into detail with this part.  I'll just mention that I had a minimal install of CentOS that was fully patched as of the installation.  

###Install InfluxDB

I hope to make use of InfluxDB in other ways within my work -- hopefully I can write on this in the future.  To install InfluxDB, I just referenced the [online documentation](https://docs.influxdata.com/influxdb/v1.3/introduction/installation/#installation)
