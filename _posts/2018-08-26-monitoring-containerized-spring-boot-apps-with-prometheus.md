---
layout: default
title: Monitoring Containerized Spring Boot Applications with Prometheus
tags: prometheus
categories: devops monitoring
---

This article discusses integrating an _already-instrumented_ Spring-Boot 2.x Java app with Prometheus. The app in question is Dockerized and exposes a Prometheus endpoint. The Prometheus service itself runs as a container and retrieves metrics from the Spring-Boot app.

Read after the break. 

<!--more-->

### Create a Petclinic Dockerfile

> We use the Petclinic sample app. Specifically we use an already-instrumented JAR present here: [v2.0.0-BUILD-SNAPSHOT](https://github.com/savishy/spring-petclinic/releases/tag/v2.0.0-BUILD-SNAPSHOT).

1. Create a new directory, say `dockerworkspace` 
1. Create the following `Dockerfile`.

```Dockerfile

# JDK application
FROM openjdk:8u131-jdk-alpine

# Labels allow for metadata and visibility
LABEL com.poc.maintainer="Vish"
LABEL com.poc.description="Petclinic Application JAR"

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

The above Dockerfile requires an `application.properties` alongside it. We use the same default application.properties from the Petclinic source. 

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
```

### Create a Petclinic Docker Image

```
$ cd dockerworkspace
$ docker build -t petclinic .
```

```
Sending build context to Docker daemon  4.096kB
Step 1/9 : FROM openjdk:8u131-jdk-alpine
 ---> a99736768b96
Step 2/9 : LABEL com.poc.maintainer="Vish"
 ---> Using cache
 ---> f26b0694ae0a
Step 3/9 : LABEL com.poc.description="Petclinic Application JAR"
 ---> Using cache
 ---> e3ac4b87e588
Step 4/9 : VOLUME /app
 ---> Using cache
 ---> 74ce95d37142
Step 5/9 : ADD https://github.com/savishy/spring-petclinic/releases/download/v2.0.0-BUILD-SNAPSHOT/spring-petclinic-2.0.0.BUILD-SNAPSHOT.jar /app/petclinic.jar
Downloading [==================================================>]   38.8MB/38.8MB
 ---> fc8430d68a7a
Step 6/9 : ADD application.properties /app/application.properties
 ---> 25dbebc07b2c
Step 7/9 : ENV JAVA_OPTS=""
 ---> Running in 0ab000bfd364
Removing intermediate container 0ab000bfd364
 ---> 3e1d861af4e0
Step 8/9 : ENV SPRING_OPTS="--spring.config.location=/app/application.properties"
 ---> Running in a6d3818cceed
Removing intermediate container a6d3818cceed
 ---> c020bf8980cb
Step 9/9 : ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /app/petclinic.jar $SPRING_OPTS" ]
 ---> Running in bc4b76359f7d
Removing intermediate container bc4b76359f7d
 ---> e5476165fb4d
Successfully built e5476165fb4d
Successfully tagged petclinic:latest
```

### Run the Petclinic App!

```
~/dockerworkspace$ docker run -p 8080:8080 -d petclinic
02fe5f19590bad333192eae00b9bf3bdc4db086ea5322e62f5de3596975dc468
```

Verify you can load the Prometheus metrics:

```
$ curl http://localhost:8080/manage/prometheus
# HELP system_load_average_1m The sum of the number of runnable entities queued to available processors and the number of runnable entities running on the available processors averaged over a period of time
# TYPE system_load_average_1m gauge
system_load_average_1m 0.0732421875
# HELP tomcat_servlet_request_max_seconds
# TYPE tomcat_servlet_request_max_seconds gauge
tomcat_servlet_request_max_seconds{name="default",} 0.0
# HELP tomcat_sessions_expired_total
# TYPE tomcat_sessions_expired_total counter
```


