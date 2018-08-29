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

### What I want to achieve

* I want to create a Prometheus Docker container on Windows with Ansible.
* As of date (Ansible 2.5.3) the `docker_container` module does not support containers on Windows. 


### Getting Started

* To start with, read the [official document](https://docs.ansible.com/ansible/2.6/dev_guide/developing_modules_general_windows.html) on custom module development.
* I will be creating a custom module within my role. For this I create a `roles/<role-name>/library` directory.
* The module I create is called `win_docker_container` so I create a file `win_docker_container.ps1`.
* I took the basic `win_environment` module from the above link and pasted that into the above file and started modifying it accordingly.

First I added a simple task that shows the intended usage of my module:

{% highlight yaml %}
- win_docker_container:
    name: prometheus_container
    network: monitoring
    state: running
    image: stefanscherer/prometheus-windows:2.2.0
{% endhighlight %}

I then created a simple `hello world` starting point as follows. Note that I have edited the `win_environment` module from above link and added appropriate variable parsing.

{% highlight powershell %}

#!powershell

# Copyright: (c) 2015, Jon Hawkesworth (@jhawkesworth) <figs@unity.demon.co.uk>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

#Requires -Module Ansible.ModuleUtils.Legacy

$ErrorActionPreference = "Stop"

$params = Parse-Args -arguments $args -supports_check_mode $true
$check_mode = Get-AnsibleParam -obj $params -name "_ansible_check_mode" -type "bool" -default $false
$diff_mode = Get-AnsibleParam -obj $params -name "_ansible_diff" -type "bool" -default $false

$name = Get-AnsibleParam -obj $params -name "name" -type "str" -failifempty $true
$state = Get-AnsibleParam -obj $params -name "state" -type "str" -default "running" -validateset "absent","present","running"
$network = Get-AnsibleParam -obj $params -name "network" -type "str"
$image = Get-AnsibleParam -obj $params -name "image" -type "str" -failifempty $true

$result = @{
    changed = $false
}

Exit-Json -obj $result

{% endhighlight %}


Running the module now gives me the message:

```
TASK [prometheus-grafana : win_docker_container] ******************************************************************************************************************************************************************
EXEC (via pipeline wrapper)
ok: [TestVMWin] => {
    "changed": false
}

```
