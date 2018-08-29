---
layout: default
title: Creating Custom Ansible Modules for Windows
tags: ansible windows
categories: devops provisioning
---

Although Ansible has great support overall, it still comes up lacking when you try to do "cutting edge" work, especially around Windows. 

In recent times, I have been working a lot with Windows Server. 
I felt the need to create my own custom modules to get certain things to work, and this was quite an interesting experience. So here's my brain dump.

<!--more-->

----

### Getting Started

* To start with, read the [official document](https://docs.ansible.com/ansible/2.6/dev_guide/developing_modules_general_windows.html) on custom module development.
* I will be creating a custom module within my role. For this I create a `roles/<role-name>/library` directory.
* The module I create is called `win_docker_container` so I create a file `win_docker_container.ps1`.
* I took the basic `win_environment` module from the above link and pasted that into the above file and started modifying it accordingly.

{% highlight ruby %}
- win_docker_container:
    name: prometheus_container
    network: '{{mon_docker_network_name}}'
    state: running
    image: '{{prom_image}}'
{% endhighlight %}
