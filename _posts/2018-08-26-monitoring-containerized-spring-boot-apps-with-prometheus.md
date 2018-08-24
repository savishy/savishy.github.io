---
layout: default
title: Monitoring Containerized Spring Boot Applications with Prometheus
tags: prometheus
categories: devops monitoring
---

This article discusses integrating a Spring-Boot 2.x Java app with Prometheus. The app in question needs to already be instrumented with the Prometheus libraries.

As discussed [in a previous post,]({% post_url 2018-08-24-spring-boot-instrumentation-with-prometheus %}), Petclinic has been instrumented to provide Prometheus metrics at a `/manage/prometheus` endpoint.

In this article we will run the instrumented Petclinic application as a Docker container. We will then run a Prometheus service (again as a Docker container) retrieve metrics from the Spring-Boot app to display in the Prometheus dashboard.

Read more after the break. 

<!--more-->

----

### Create a Petclinic Dockerfile

> We use the Petclinic sample app that was previously instrumented. The instrumented JAR is present here: [v2.0.0-BUILD-SNAPSHOT](https://github.com/savishy/spring-petclinic/releases/tag/v2.0.0-BUILD-SNAPSHOT).
> Some knowledge of Docker and a working Docker installation is advised.

1. Create a new "Docker Workspace" directory for dockerizing Petclinic, say `dockerpetclinic`.
1. Create the following `Dockerfile`.

```Dockerfile

# Base image that contains OpenJDK 8.0
FROM openjdk:8u131-jdk-alpine

# Labels allow for metadata and visibility
LABEL com.poc.maintainer="Vish"
LABEL com.poc.description="Petclinic Application JAR"

# Persistence for the application directory.
VOLUME /app

# Application and configuration
ADD https://github.com/savishy/spring-petclinic/releases/download/v2.0.0-BUILD-SNAPSHOT/spring-petclinic-2.0.0.BUILD-SNAPSHOT.jar /app/petclinic.jar
ADD application.properties /app/application.properties
ENV JAVA_OPTS=""

# Spring app needs to take local application.properties
ENV SPRING_OPTS="--spring.config.location=/app/application.properties"
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /app/petclinic.jar $SPRING_OPTS" ]

```


### Create an application.properties

The above Dockerfile requires an `application.properties` alongside it. We use the same default application.properties from the Petclinic source with one modification: `server.port=9080` to allow Petclinic to listen on a non-default port. 

```
# database init, supports mysql too
database=hsqldb
spring.datasource.schema=classpath*:db/${database}/schema.sql
spring.datasource.data=classpath*:db/${database}/data.sql

# Web
spring.thymeleaf.mode=HTML

# JPA
spring.jpa.hibernate.ddl-auto=none

# Internationalization
spring.messages.basename=messages/messages

# Actuator / Management
management.endpoints.web.base-path=/manage
management.endpoints.web.exposure.include=*

# Logging
logging.level.org.springframework=INFO
# logging.level.org.springframework.web=DEBUG
# logging.level.org.springframework.context.annotation=TRACE

# Maximum time static resources should be cached
spring.resources.cache.cachecontrol.max-age=12h

server.port=9080
```

The key piece in this is:

```
management.endpoints.web.base-path=/manage
management.endpoints.web.exposure.include=*
```

This means all the management endpoints are enabled. This has an impact on security so be prudent about exposing management endpoints.

### Create a Petclinic Docker Image

Build an image named `petclinic`:

```
$ cd dockerpetclinic
$ docker build -t petclinic .
```

Sample Output:

```
...
...
Step 5/9 : ADD https://github.com/savishy/spring-petclinic/releases/download/v2.0.0-BUILD-SNAPSHOT/spring-petclinic-2.0.0.BUILD-SNAPSHOT.jar /app/petclinic.jar
Downloading [==================================================>]   38.8MB/38.8MB
 ---> fc8430d68a7a
...
...
Step 9/9 : ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /app/petclinic.jar $SPRING_OPTS" ]
 ---> Running in bc4b76359f7d
...
...
Successfully built e5476165fb4d
Successfully tagged petclinic:latest
```

### Run the Petclinic App!

```
docker run -p 9080:9080 -d petclinic
```

Verify you can load the Prometheus metrics. Open `http://localhost:9080/manage/prometheus`.

```
$ 
# HELP system_load_average_1m The sum of the number of runnable entities queued to available processors and the number of runnable entities running on the available processors averaged over a period of time
# TYPE system_load_average_1m gauge
system_load_average_1m 0.0732421875
# HELP tomcat_servlet_request_max_seconds
# TYPE tomcat_servlet_request_max_seconds gauge
tomcat_servlet_request_max_seconds{name="default",} 0.0
# HELP tomcat_sessions_expired_total
# TYPE tomcat_sessions_expired_total counter
```

### Create a Prometheus Docker Image

Prometheus already has a Docker Image [here](https://hub.docker.com/r/prom/prometheus/). This is the official image, but needs minor customization for our use-case.
We need to configure Prometheus to read the metrics from our Petclinic app.

> Note: The Prometheus container usually runs in the same server as the application container. Also, Prometheus and the application are launched within the same `docker network`. 

1. First create a new docker workspace `mkdir ~/dockerprometheus`
1. Create a YAML file `prometheus.yml`:

```
global:
  scrape_interval: 10s

scrape_configs:
- job_name: 'petclinic-app'
  metrics_path: /manage/prometheus/
  static_configs:
  - targets: ['petclinic:9080']
```

1. Next, create a `Dockerfile`. This will add our YAML file to the Prometheus base image.

```
FROM prom/prometheus:v2.3.2
COPY prometheus.yml /etc/prometheus/prometheus.yml
```

Let's build a Docker Image named `prometheus`.

```
cd ~/dockerprometheus
docker build -t prometheus .
```

### Run Docker Containers!

> Notice that the `prometheus.yml` references the `targets` as `petclinic:9080` i.e. it will look for a DNS entry `http://prometheus`.
> This is possible only if we create a bridge network and place both our Petclinic and Prometheus containers in it.

First create a bridge network named `dev`:

```
docker network create dev
```

Create a `prometheus` and a `petclinic` container from the images created above. 
Place both containers in the `dev` network.

```
docker run -p 9080:9080 -d --net dev --name petclinic petclinic
docker run -d -p 9090:9090 --net dev --name prometheus prometheus
```

### Verify

Look at the Prometheus service dashboard. You should see one target with a state of "UP".
