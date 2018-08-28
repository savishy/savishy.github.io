---
layout: default
title: Visualizing Prometheus Metrics for Spring Boot Apps with Grafana
tags: prometheus grafana
categories: devops monitoring
---


Spring Boot Dashboard for Grafana:
https://grafana.com/dashboards/6756

Import the Dashboard into Grafana:

![image](https://user-images.githubusercontent.com/13379978/44720520-b7ef9f80-aae4-11e8-8f2c-f7e4c852b95e.png)

Make some fake requests to Petclinic:

```
$ while true; do curl -X GET http://some.ip.address:9080/owners?lastName=; sleep 1; done
```

Watch the trace show up in Grafana!

![image](https://user-images.githubusercontent.com/13379978/44719363-b4f2b000-aae0-11e8-874a-fdbf63d27d21.png)
