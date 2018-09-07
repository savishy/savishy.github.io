---
layout: default
title: Creating Immutable yet Configurable Docker Images
tags: docker 
categories: devops containerization
---

One challenge when creating Dockerized apps is: 

> How do you create images that retain their immutability while remaining configurable across environments?

<!--more-->

The solution I like to propose is based on the following key ideas:

> ### :bulb: Dockerized apps _are artifacts_ just like the application artifact packaged within them. 

For your application's artifact (e.g. petclinic.jar) you would use strategies such as build-once-deploy-anywhere. 
This means you avoid building the application more than once and deploy the same artifact across environments.

You need to treat Docker images as artifacts as well - they need to built _only once_. 

> ### :bulb: Configuration forms the "mutable" part of a Docker image.

The app packaging, JRE version, libraries and other dependencies form the "immutable" aspect of a Docker image.
The mutable component is the configuration.
