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


### 1. Getting Started

* To start with, read the [official document](https://docs.ansible.com/ansible/2.6/dev_guide/developing_modules_general_windows.html) on custom module development.
* I will be creating a custom module within my role. For this I create a `roles/<role-name>/library` directory.
* The module I create is called `win_docker_container` so I create a file `win_docker_container.ps1`.
* I took the basic `win_environment` module from the above link and pasted that into the above file and started modifying it accordingly.
* I have a Windows Server 2016 VM on Azure which I used for testing. The Azure VM is called "Windows Server 2016 Datacenter - with Containers". 

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

# INSERT CODE HERE


# END INSERT
Exit-Json -obj $result

{% endhighlight %}


Running the module on a Windows Server 2016 VM now gives me the message:

```
TASK [prometheus-grafana : win_docker_container] ******************************************************************************************************************************************************************
EXEC (via pipeline wrapper)
ok: [TestVMWin] => {
    "changed": false
}
```

Now let's start developing the module!

----

### 2. Add Basic logic to create containers declaratively

My logic will include the following for starters:

* Check whether a container exists with the given name.
* If yes, do not create a new container. Return existing container's ID.
* If no, create a container. Return new container's ID.

The logic would look something like this. I added this in between the areas marked `INSERT CODE HERE`. 

{% highlight powershell %}

$existingContainers = $(docker ps -aq --filter "name=$($name)")
if ($existingContainers -ne $null) {
    # existing containers with the same name
} else {
    # no existing containers; create
    $result.changed = $true
}

{% endhighlight %}


----

### 3. Add error handling and ability to handle containers without custom networks.

In this iteration I added some more capability:

1. If a `network` argument is passed the module will check if the docker network already exists.
1. If it does not, the module fails with a user-friendly error.
1. If containers already exist with given name, the container's ID is returned.
1. If containers do not exist, logic is available to create them with or without a custom network.

{% highlight powershell %}

if ($network -ne $null) {
    $networks = $(docker network ls -q --filter "name=$($network)")
    if ($networks -eq $null) {
        Fail-Json -obj $result -message "When a network name is provided, create the network before calling this module."
    }
}
$existingContainers = $(docker ps -aq --filter "name=$($name)")
if ($existingContainers -ne $null) {
    # existing containers with the same name
    $result.container_id = $existingContainers
} else {
    # no existing containers; create

    if ($network -ne $null) {
        $newContainer = $(docker run --net "$($network)" --name "$($name)" -d "$($image)")
    } else {
        $newContainer = $(docker run --name "$($name)" -d "$($image)")
    }

    $result.changed = $true
    $result.container_id = $newContainer
}


{% endhighlight %}

And the output of this is:

```
changed: [TestVMWin] => {
    "changed": true,
    "container_id": "c300ba5743da877eea1f1ef129b135725ce6fa52beb32c19c0860bb068e62b73"
}
```

The true test of declarativeness would come when I ran it twice - so I did, and here's the output :+1:

```
ok: [TestVMWin] => {
    "changed": false,
    "container_id": "c300ba5743da"
}
```
----

### 4. Add Port Forwarding

In this iteration, I:

1. Added an additional parameter `publish_all_ports` which publishes all ports marked using the `EXPOSE` keyword. The default value for this is `true`.
1. Refactored the way we invoke the command.

The invocation of this module now changes to:

{% highlight yaml %}
- win_docker_container:
    name: prometheus_container
    network: 'nat'
    state: running
    image: 'stefanscherer/prometheus-windows:2.2.0'
    publish_all_ports: true
{% endhighlight %}

### Notes

* Port Forwarding seems to work only with the `nat` network created by default. Adding new networks does not seem to work with Docker on Windows. (In other words I have a little more understanding left for Docker on Windows)
* The web service is exposed on a random port which we would need to find out by executing `docker ps -a` on the Docker Host.
* Accessing the web service is possible through the IP of the virtual network - `localhost` does not work.


### 5. The complete module

{% highlight powershell %}

$ErrorActionPreference = "Stop"

$params = Parse-Args -arguments $args -supports_check_mode $true
$check_mode = Get-AnsibleParam -obj $params -name "_ansible_check_mode" -type "bool" -default $false
$diff_mode = Get-AnsibleParam -obj $params -name "_ansible_diff" -type "bool" -default $false

$name = Get-AnsibleParam -obj $params -name "name" -type "str" -failifempty $true
$state = Get-AnsibleParam -obj $params -name "state" -type "str" -default "running" -validateset "absent","present","running"
$network = Get-AnsibleParam -obj $params -name "network" -type "str"
$image = Get-AnsibleParam -obj $params -name "image" -type "str" -failifempty $true
$publish_all_ports = Get-AnsibleParam -obj $params -name "publish_all_ports" -type "bool" -default $true

$result = @{
    changed = $false
}

if ($network -ne $null) {
    $networks = $(docker network ls -q --filter "name=$($network)")
    if ($networks -eq $null) {
        Fail-Json -obj $result -message "When a network name is provided, create the network before calling this module."
    }
}
$existingContainers = $(docker ps -aq --filter "name=$($name)")
if ($existingContainers -ne $null) {
    # existing containers with the same name
    $result.container_id = $existingContainers
} else {
    # no existing containers; create
    $networkCmd = if ($network -ne $null) { "--net $($network)" } else { "" }
    $portsCmd = if ($publish_all_ports) { "-P" } else { "" }

    $command = "docker run $($networkCmd) $($portsCmd) --name $($name) -d $($image)"
    $newContainer = iex $command

    $result.command = $command
    $result.changed = $true
    $result.container_id = $newContainer
}


Exit-Json -obj $result


{% endhighlight %}


The final output:

```
changed: [TestVMWin] => {
    "changed": true,
    "command": "docker run --net nat -P --name prometheus_container -d stefanscherer/prometheus-windows:2.2.0",
    "container_id": "042c6aafd50fb74426b774533b5d3a687b292181af8ac8383b848faffb2d361a"
}
```
