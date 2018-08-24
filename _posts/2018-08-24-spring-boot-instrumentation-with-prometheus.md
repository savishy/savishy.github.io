---
layout: post
title: Instrumenting Spring Boot Applications for Prometheus Monitoring
tags: prometheus
categories: 
- devops
- software
---

This post talks about instrumenting a sample Spring-Boot 2.0 application with Prometheus. The motivation to write this post came from my experiments toward creating an instrumented, Dockerized Spring-Boot app that would generate metrics to be consumed by Prometheus and Grafana. Read more after the break.

<!--more-->

### The Sample App

We use the Spring Boot sample application [Petclinic](https://github.com/spring-projects/spring-petclinic) for this exercise. 

A default `application.properties` is loaded with the application, present [here](https://github.com/spring-projects/spring-petclinic/blob/master/src/main/resources/application.properties).

Also note that by default the application uses an in-memory HSQL DB. All this information is useful to us. 

### Add Micrometer Prometheus to your POM

> The great thing is, enabling Prometheus is as simple as dropping-in a single dependency to your POM. No code changes are needed! :+1:

1. Checkout the repository.
1. Add the following to the pom.xml. More at [https://micrometer.io/docs/installing](https://micrometer.io/docs/installing). 
1. (here we are using 1.0.6 as the version for the dependency):

```
<dependencies>
...
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.0.6</version>
  </dependency>
```

### Build the Petclinic application.

1. Build the JAR as per the instructions (TL;DR: `./mvnw install`). Note that you would need JDK 8.0, and `JAVA_HOME` must be set.
1. This creates `target/spring-petclinic-<version>.jar`. 

### Create `application.properties`. 

To instrument the application we will provide an "overrided" application.properties, so Petclinic will take "our" properties file. 

1. First create a directory `C:\app`. 
1. Create a text file `application.properties`. 
1. Copy all the properties from the original file [here](https://github.com/spring-projects/spring-petclinic/blob/master/src/main/resources/application.properties) into our file.
1. Add _one additional line_: `server.port = 9080`. We are overriding the default port to be 9080.
1. Copy the `target/spring-petclinic-<version>.jar` file into `C:\app`. 

### Run the Petclinic App with the `application.properties`.

```
C:\>java -jar C:\app\spring-petclinic-2.0.0.BUILD-SNAPSHOT.jar --spring.config.location=C:\app\application.properties
```

Validate that Petclinic starts, and is using our application.properties. You should see the line below that shows the embedded Tomcat has started on port 9080.

```
2018-08-24 11:46:00.371  INFO 16336 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 9080 (http) with context path ''
2018-08-24 11:46:00.371  INFO 16336 --- [           main] o.s.s.petclinic.PetClinicApplication     : Started PetClinicApplication in 8.25 seconds (JVM running for 8.994)
```

You can now browse to `http://localhost:9080` to view the Petclinic app.
Hit Ctrl-C on the command prompt at any time to stop the running application. 

### Expose the Prometheus Endpoints!

Now for the fun part. Let's expose the Prometheus endpoint, i.e. enable viewing Prometheus metrics from a `localhost:9080/.....` URL.

We will expose 3 endpoints: `info`, `health` and `prometheus`. :exclamation: _these endpoints may expose sensitive information so be careful not to expose them in production applications._

```
management.endpoint.prometheus.enabled=true
management.endpoints.web.exposure.include = info, health, prometheus
```

([Read more about exposing management endpoints.](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints))


## References:

1. https://prometheus.io/docs/practices/instrumentation/
1. https://docs.spring.io/spring-metrics/docs/current/public/prometheus#timers
1. https://dzone.com/articles/using-micrometer-with-spring-boot-2
1. https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints
1. http://www.springboottutorial.com/spring-boot-application-configuration
