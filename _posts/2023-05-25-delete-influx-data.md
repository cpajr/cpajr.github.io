---		
title: 'Deleting data from Influx DB'
classes: wide
---
This post is mostly to remind me of the command to delete data from Influx DB 1.8 that is older than a certain date:
```
delete from "switch" where time < '2022-04-25'
```