---
date: 2018-08-22
layout: post
title: 'Cisco Command Authorization with Tacacs+'
---

Oh, the struggles one has with maintaining the AAA: authentication, authorization, and accounting.  While migrating to a new authentication server, we realized that our limited command set for a group of users was not working correctly.  After doing some research, we realized there were some missing commands from our configuration.

The hassle is that we want a certain group of users to have level 15 access while not allowing configuration authority.  To provide this, we leverage the command authorization in Tacacs+ and (the most important part) having a proper switch configuration.  

The following is an example of our AAA configuration:

```
aaa new-model
aaa authentication login default local group tacacs+
aaa authorization console
aaa authorization config-commands
aaa authorization exec default group tacacs+ local
aaa authorization commands 15 default local group tacacs+
aaa accounting exec default start-stop group tacacs+
aaa accounting commands 15 default start-stop group tacacs+
aaa accounting connection default start-stop group tacacs+
aaa accounting system default start-stop group tacacs+
aaa session-id common
```
The key command for our particular goal is the `aaa authorization commands 15 default local group tacacs+`.  Lesson learned for the day.  

Now, my next task is to do the same for Nokia routers.  