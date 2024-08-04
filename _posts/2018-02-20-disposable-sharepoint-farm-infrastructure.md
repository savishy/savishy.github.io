# Disposable SharePoint Farm Infrastructure

* _Dates:_ Feb 2018 to Aug 2018
* _Role:_ Architect, Team Lead
* _Team Size:_ 5

## Project Goals

Our 5-member team was deployed to our client's site with the brief to design, implement and deliver a fully-automated SharePoint Server Farm on Azure.

Infrastructure-as-Code was a key deliverable, i.e. the following tasks were fully automated:
1. Networks Provisioning
1. Provisioning of all VMs, Disks, NICs, Security Groups and related resources in Azure
1. Configuring SQL Server Databases and High Availability
1. SharePoint Services and Web-Apps Configuration
1. Configuration of firewall rules for Barracuda WAF and Azure Traffic Manager
1. Infrastructure Validation against compliance requirements.

### Context
* The client used Sharepoint Server for many of their critical business processes, and were embarking on an Azure migration.
* Their existing provisioning process was highly manual, involving 48+ hours of manual provisioning by a consultant.
* End-to-end automation would have the potential to save time, cost (consultant fees) and minimize risk due to human error.

## Tools Used

<img src="https://user-images.githubusercontent.com/13379978/43992448-dbe8249e-9d9b-11e8-8245-6d8a2f9f131a.png" alt="TeamCity" height="50"/><img src="https://user-images.githubusercontent.com/13379978/43992486-753b8082-9d9c-11e8-9e89-c7426ad7e0e3.png" alt="Ansible" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992501-a738dc60-9d9c-11e8-8450-3bd1ec0dcff1.png" alt="Inspec" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992517-f6abf908-9d9c-11e8-9364-734c9892e2ce.png" alt="Octopus" height="50"/>

<img src="https://user-images.githubusercontent.com/13379978/43992539-63d55376-9d9d-11e8-9040-866efa5ce79d.png" alt="Azure" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992556-a07bcb34-9d9d-11e8-9460-a00788b20e56.png" alt="PowerShell DSC" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992565-dda1f286-9d9d-11e8-84ac-9f55b9238dcc.png" alt="SharePoint 2016" height="50"/> <img src="https://user-images.githubusercontent.com/13379978/43992572-f208da78-9d9d-11e8-9dd7-a186fab797aa.png" alt="SQL Server 2016" height="50"/>

## Key Activities

### Drove Recruiting

The team was hired and upskilled in anticipation of this project. As a senior team-member, I helped drive the recruiting, interviewing and upskilling activities. Given that Agility Roots was a new firm, I also created a bespoke interviewing process and programming assignment to help narrow down the most suitable candidates. 

### Upskilled the Team

Once the team was recruited and in place, I identified gaps in their skillsets and conducted weekend hacking sessions to upskill the team, as well as build camaraderie. This went a long way toward our collective success in delivering a complex project.

## Key Impact

* Designed and implemented a fully automated process to deliver an HA SharePoint Farm on Azure.
* Delivered 10 environments with 70 servers and a high degree of reliability.
* The entire process from scratch took only 8 hours per environment.
