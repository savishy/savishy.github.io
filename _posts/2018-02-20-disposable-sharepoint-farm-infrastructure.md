# Disposable SharePoint Farm Infrastructure

* _Dates:_ Feb 2018 to Aug 2018
* _Role:_ Architect, Team Lead
* _Team Size:_ 5

## Project Goals

Our team was tasked with the design, implementation and delivery of a fully-automated SharePoint Server Farm on Azure.

Infrastructure-as-Code was a key deliverable, i.e. the following tasks were fully automated:
1. Networks Provisioning
1. Provisioning of all VMs, Disks, NICs, Security Groups and related resources in Azure
1. Configuring SQL Server Databases and High Availability
1. SharePoint Services and Web-Apps Configuration
1. Configuration of firewall rules for Barracuda WAF and Azure Traffic Manager
1. Infrastructure Validation against compliance requirements.

## Tools Used

<img src="https://user-images.githubusercontent.com/13379978/43992448-dbe8249e-9d9b-11e8-8245-6d8a2f9f131a.png" alt="TeamCity" height="50"/><img src="https://user-images.githubusercontent.com/13379978/43992486-753b8082-9d9c-11e8-9e89-c7426ad7e0e3.png" alt="Ansible" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992501-a738dc60-9d9c-11e8-8450-3bd1ec0dcff1.png" alt="Inspec" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992517-f6abf908-9d9c-11e8-9364-734c9892e2ce.png" alt="Octopus" height="50"/>

<img src="https://user-images.githubusercontent.com/13379978/43992539-63d55376-9d9d-11e8-9040-866efa5ce79d.png" alt="Azure" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992556-a07bcb34-9d9d-11e8-9460-a00788b20e56.png" alt="PowerShell DSC" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992565-dda1f286-9d9d-11e8-84ac-9f55b9238dcc.png" alt="SharePoint 2016" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992572-f208da78-9d9d-11e8-9dd7-a186fab797aa.png" alt="SQL Server 2016" height="50"/>

## Key Impact

* Designed and implemented a fully automated process to deliver an HA SharePoint Farm on Azure.
* Delivered 10 environments with 70 servers and a high degree of reliability.
* The entire process from scratch took only 8 hours per environment.
