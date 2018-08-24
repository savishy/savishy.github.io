---
layout: default
title: Monitoring Containerized Spring Boot Applications with Prometheus
tags: prometheus
categories: devops monitoring
---

This article discusses integrating an _already-instrumented_ Spring-Boot 2.x Java app with Prometheus. The app in question is Dockerized and exposes a Prometheus endpoint. The Prometheus service itself runs as a container and retrieves metrics from the Spring-Boot app.

Read after the break. 

<!--more-->

### Containerize Petclinic

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
