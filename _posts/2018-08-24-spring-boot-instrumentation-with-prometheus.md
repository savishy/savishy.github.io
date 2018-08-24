---
layout: default
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



{% highlight xml %}
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.0.6</version>
  </dependency>
{% endhighlight %}

> Here we are using 1.0.6 as the version for the dependency. Check if a later version is available.


### Build the Petclinic application JAR.

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

### Expose the Prometheus Endpoints.

Now for the fun part. Let's expose the Prometheus endpoint, i.e. enable viewing Prometheus metrics from a `localhost:9080/.....` URL.

We will expose 3 endpoints: `info`, `health` and `prometheus`. 

> _These endpoints may expose sensitive information so be careful not to expose them in production applications._

Add the following to your application.properties:

```
management.endpoint.prometheus.enabled=true
management.endpoints.web.exposure.include = info, health, prometheus
```

> Read more about [exposing management endpoints.](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints)

### Rerun the application with instrumentation!

In the previous step we changed the application.properties. This does not require recompilation. Simply execute the `java -jar` command again. 

Monitor the logs printed to the console. You should see this:

```
2018-08-24 12:16:12.887  INFO 15092 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/manage'
```

This means endpoints have been exposed, and you need to prefix them with `/manage`.

```

2018-08-24 12:41:45.104  INFO 8360 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 3 endpoint(s) beneath base path '/manage'
2018-08-24 12:41:45.120  INFO 8360 --- [           main] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped "{[/manage/health],methods=[GET],produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping$OperationHandler.handle(javax.servlet.http.HttpServletRequest,java.util.Map<java.lang.String, java.lang.String>)
2018-08-24 12:41:45.120  INFO 8360 --- [           main] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped "{[/manage/info],methods=[GET],produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping$OperationHandler.handle(javax.servlet.http.HttpServletRequest,java.util.Map<java.lang.String, java.lang.String>)
2018-08-24 12:41:45.120  INFO 8360 --- [           main] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped "{[/manage/prometheus],methods=[GET],produces=[text/plain;version=0.0.4;charset=utf-8]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping$OperationHandler.handle(javax.servlet.http.HttpServletRequest,java.util.Map<java.lang.String, java.lang.String>)
2018-08-24 12:41:45.120  INFO 8360 --- [           main] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped "{[/manage],methods=[GET],produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}" onto protected java.util.Map<java.lang.String, java.util.Map<java.lang.String, org.springframework.boot.actuate.endpoint.web.Link>> org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping.links(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
```

These lines give you a clue as to what endpoints to hit.

* `http://localhost:9080/manage/health` provides the health endpoint
* `http://localhost:9080/manage/info` provides the info endpoint
* `http://localhost:9080/manage/prometheus` provides the prometheus endpoint


## References:

1. https://prometheus.io/docs/practices/instrumentation/
1. https://docs.spring.io/spring-metrics/docs/current/public/prometheus#timers
1. https://dzone.com/articles/using-micrometer-with-spring-boot-2
1. https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints
1. http://www.springboottutorial.com/spring-boot-application-configuration
