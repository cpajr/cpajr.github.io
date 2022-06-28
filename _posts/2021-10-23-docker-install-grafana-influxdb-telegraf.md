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
    - /projects/monitoring/telegraf/mibs:/usr/share/snmp/mibs
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
    - /projects/monitoring/grafana/etc/grafana/grafana.ini:/etc/grafana/grafana.ini
```

Allow me to give some explanation to each section and what is being accomplished.

## InfluxDB 

There isn't much to share under this section.  I mount the `grafana.ini` through a volume so that unique configurations could be made; however, in my deployment, I decided to only keep the default configuration.  At this time, I decided not to encrypt communication to and from the InfluxDB.

## Telegraf

There are a couple of things to note here for Telegraf

### SNMP MIBS

For my purpose, I am polling mostly network devices and, as of today, the only good way to get data from them is via SNMP[^1].  However, using the default Docker container for Telegraf does not include the needed SNMP MIBS.  To overcome this, I pulled together my own group of needed MIBS and mounted it as a volume to the container.  

### Telegraf Configs

There are numerous configurations available for Telegraf, but for an initial deployment there is only one change: Outputs.  In the telegraf.conf file, find the section for `outputs.influxdb`:

  * Update `urls` to `urls = ["http://influxdb:8086"]`.  This will direct to the InfluxDB container
  * Update `database` to the desire database within InfluxDB.  In this case, I updated it to `database = "telegraf"`

```
[[outputs.influxdb]]
 
  urls = ["http://influxdb:8086"]
  database = "telegraf"
```

## Grafana

In my particular deployment, I wanted for the authentication to Grafana be done via SAML; particularly, I needed authentication to be done through Azure AD.  Thankfully, Grafana has already built-in the needed items for this to occur and [provided the needed documentation](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/azuread/).  The following are the unique changes that I made:

```
[server]
# The public facing domain name used to access grafana from a browser
domain = monitor.cpajr.com

# The full public facing url you use in browser, used for redirects and emails
# If you use reverse proxy and sub path specify full url (with sub path)
root_url = https://monitor.cpajr.com

[auth]
# Set to true to disable (hide) the login form, useful if you use OAuth, defaults to false
disable_login_form = true

#################################### Azure AD OAuth #######################
[auth.azuread]
name = Azure AD
enabled = true
allow_sign_up = true
client_id = <Azure Client ID>
client_secret = <Azure Client Secret>
scopes = openid email profile
auth_url = https://login.microsoftonline.com/<Tenant ID>/oauth2/v2.0/authorize
token_url = https://login.microsoftonline.com/<Tenant ID>/oauth2/v2.0/token
allowed_domains =
allowed_groups =
```

With this all up and running, you can continue to follow the previous article I wrote.  

Enjoy!

[^1]: Cisco now provides a way to stream Telemetry from their 9200 and 9300 series switches.  There are plugins available into Telegraf which allows for this data to be streamed from the devices.  However, in my inital testing, I found it to be unreliable.  For further details see [this link](https://blogs.cisco.com/developer/getting-started-with-model-driven-telemetry)    