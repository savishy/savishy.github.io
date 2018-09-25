---
layout: default
title: Creating Dynamic Inventory with Ansible Tower
tags: ansible awx 
categories: devops infrastructure
---

Ansible's cloud inventory scripts are well-known for their ability to manage your cloud environment without hardcoding IPs anywhere. 
This article helps understand how to manage the same inventory using Ansible Tower.

<!--more-->

**AWX (or Ansible Tower Free Version)**

Ansible has been working on a community alternative to Ansible Tower, called [AWX](https://github.com/ansible/awx) - it provides the same backend functionality as Ansible Tower while being free and open-source.

Obviously there are caveats though. The installation of AWX is done _using Ansible itself_ (don't get confused). Essentially:

1. You log in to the server where you want to run AWX
1. Checkout the AWX Github repository
1. Run the installer from [the `installer` directory](https://github.com/ansible/awx/tree/devel/installer).
  `ansible-playbook -i inventory install.yml`

I won't go into the rest of the process as it is well documented in the Ansible Tower documentation. 

So here's how you manage dynamic inventory with AWX.

**Step 1: Credential**

Create a Credential that stores your Azure Service Principal Details.

![image](https://user-images.githubusercontent.com/13379978/45993611-530c7280-c0ad-11e8-8911-25ef48d3572f.png)

**Step 2: Dynamic Inventory Script**

Copy the contents of [azure_rm.py](https://github.com/ansible/ansible/blob/devel/contrib/inventory/azure_rm.py) from Ansible's repository.

![image](https://user-images.githubusercontent.com/13379978/45993835-623ff000-c0ae-11e8-82ae-20be7df66746.png)

**Step 3: New Inventory that uses script**

Create a new inventory and save it.

![image](https://user-images.githubusercontent.com/13379978/45993876-987d6f80-c0ae-11e8-8c41-92198e13a061.png)

* Visit Sources and hit the + button to add a new source.
* Choose Source: "Custom Script"
* Choose Custom Inventory Script: "azure_rm.py" (the script you created previously)
* Choose Credentials: "DMAzure" (the credentials you created previously)

The first time you create the dynamic inventory, you need to sync it.
![image](https://user-images.githubusercontent.com/13379978/45993995-3f620b80-c0af-11e8-9a59-3fea95a1d499.png)

If the sync succeeded you would see your hosts fetched _dynamically_ from Azure :+1:

![image](https://user-images.githubusercontent.com/13379978/45994066-a1bb0c00-c0af-11e8-9153-33999cd5bb4e.png)
