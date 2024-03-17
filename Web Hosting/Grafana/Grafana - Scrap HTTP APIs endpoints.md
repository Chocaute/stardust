---
created: 2024-02-22T09:23
updated: 2024-02-22T09:27
tags:
  - Grafana
  - OVH
  - API
---
Use the "Infinity" data source plugin. 

https://grafana.com/docs/plugins/yesoreyeram-infinity-datasource/latest/

> The Infinity data source plugin allows you to query and visualize data from JSON / CSV / GraphQL / XML / HTML endpoints.

___

In memory, for OVH : In the data source configuration, use a bearer key. I didn't find another way. 

You can find a bearer key in the new version of the OVH API website. I don't recommend it for production, though. If this key was somehow leaked, someone would have complete control of the account.
