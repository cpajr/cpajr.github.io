---		
title: 'Docker Install for Grafana, InfluxDB, and Telegraf'
classes: wide
---

Several years ago, I created a post describing the process to install Grafana, InfluxDB, and Telegraf on CentOS.  I've used this combination to provide monitoring and visualiation of a network for many years.  Honestly, it has served me well.  But, as can be expected, I've grown to appreciate the flexiblity and ease of using Docker to deploy applications.  I particularly appreciate the bundled nature of a Docker application.  This post will be a follow up to my previous article, [Installing Influxdb, Telegraf, and Grafana on CentOS](https://cpajr.com/install-influxdb-telegraf-grafana/){:target="_blank" rel="noopener"}

# Outline

As stated above, this post will focus on installing Grafana, InfluxDB, and Telgraf through Docker.  Specifically, we will be installing based on the following parameters:

- Docker, version 20.10.9
- Grafana, version 8.2.2
- InfluxDB, version 1.8.10
- Telegraf, 1.20.2

It should be noted, all of the above are the latest version with the exception of InfluxDB.  I decided to not migrate, yet, to the InfluxDB 2.0. 

# Install Docker

Personally, I have found better success with using the documented [Docker installation for Ubuntu](https://docs.docker.com/engine/install/ubuntu/).  In addition to installing Docker, also install `docker-compose`:

```
# apt install docker-compose
```

To verify that Docker has been installed, issue the command `docker ps -a`.  You should see the following output:

```
# docker ps -a

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

#
```

# Prepare docker-compose

Next step is to prepare your `docker-compose.yml` file.  The following is an example of what I did for my installation:

```
version: '3.4'
services:
  influxdb:
    image: influxdb:1.8
    container_name: influxdb
    hostname: influxdb
    ports:
    - '8086:8086'
    volumes:
    - /projects/monitoring/influxdb/var/lib/influxdb:/var/lib/influxdb
    - /projects/monitoring/influxdb/etc/influxdb:/etc/influxdb
  telegraf:
    image: telegraf:latest
    container_name: telegraf
    hostname: telegraf
    depends_on:
    - influxdb
    entrypoint: '/usr/bin/telegraf --config-directory /etc/telegraf/telegraf.d'
    volumes:
    - /projects/monitoring/telegraf/etc/telegraf:/etc/telegraf
    - /var/run/docker.sock:/var/run/docker.sock
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    hostname: grafana
    depends_on:
    - influxdb
    - telegraf
    ports:
    - '127.0.0.1:3000:3000'
    volumes:
    - /projects/monitoring/grafana/var/lib/grafana:/var/lib/grafana
```

Allow me to give some explanation to each section and what is being accomplished.

## InfluxDB 

