---

copyright:
  years: 2025
lastupdated: "2025-10-24"

keywords: migration solutions, classic to vpc migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}


# Migration Solutions
{: #solutions}

Depending on your environment and workloads, you can choose from one of the different solutions to migrate to VPC. 

There is no one approach fits all when it comes to migration, your choice of solutions and tools should be driven by your classic environment complexity and your modernization goals.
{: shortdesc}

## Typical Migration Process
{: #typical-process}

Migration is a multi-step process, typically involving but not limited to

1. **Discovery**
   Discover your existing classic resources including compute, storage and network resources.

2. **Mapping to VPC**
   Map your existing environment to {{site.data.keyword.vpc_short}} landing zone, including [locations](/docs/overview?topic=overview-locations), VPCs, network, storage profiles, server profiles and OS versions.

3. **Provision your landing zone**
   Provision your base {{site.data.keyword.vpc_short}} environment including subnets, VSIs and empty storage volumes. You can use [deployable architectures](/docs/secure-enterprise?topic=secure-enterprise-understand-module-da) including deployable architecture solutions like the [{{site.data.keyword.vpc_short}} landing zone](https://cloud.ibm.com/catalog/architecture/deploy-arch-ibm-slz-vpc){ :external}.

4. **Deploy Application**
   Deploy your application and its dependencies onto the {{site.data.keyword.vpc_short}} VSI by leveraging your continous delivery pipeline.

5. **Migrate your data**
   Migrate your application data to sync your files and folders from source data volumes to target. Optionally, schedule a periodic sync to push changes.

6. **Validate**
   Validate your application in the target environment.

7. **Cutover**
   Finally, stop processing writes on source and cutover from classic to {{site.data.keyword.vpc_short}}. Plan for deprovisioning your old environment.

The various sections in the guide provides guidance, pointers and existing tools and techniques to go through your migration process in a Do-it-Yourself manner. 

If you prefer to leverage external third-party services and capabilities to perform the same, below are a few options you can evaluate.

## VPC+ Cloud Migration
{: #vpc-cloud-migration}

If you want to migrate your {{site.data.keyword.cloud_notm}} classic infrastructure (compute, network, and storage) to VPC, you can use {{site.data.keyword.vpc-plus-migration}}. {{site.data.keyword.vpc-plus-migration}} is a third-party, software-based migration-as-a-service solution, provided by Wanclouds, for migrating components from classic infrastructure to your VPC. With {{site.data.keyword.vpc-plus-migration}}, you can discover and choose resources for migration, create, and set up those resources in your VPC environment. You can also run and manage your VPC environment from within the tool.

You can migrate the following key elements of {{site.data.keyword.cloud_notm}} classic infrastructure to VPC:

* Subnets
* Virtual server instances
* Dedicated hosts
* Storage volumes (primary and secondary)
* Security groups
* Load balancers
* Firewall (ACL) configuration
* VPN configuration
* SSH keys
* Public gateway

For more information, see [Getting started with {{site.data.keyword.vpc-plus-migration}}](/docs/wanclouds-vpc-plus?topic=wanclouds-vpc-plus-getting-started-tutorial).


## ConvertIO Workload Migration
{: #convertio}

PrimaryIO empowers organizations to transition to the cloud seamlessly—whether adopting it as a primary production environment or leveraging it for disaster recovery. By offering a comprehensive suite of tools and services, PrimaryIO facilitates efficient, secure, and low-risk migration of VMware workloads to IBM Cloud, ensuring minimal disruption and maximum operational continuity.

Benefits of using PrimaryIO for IBM Cloud migration:

* Complete VM migration - managed by PrimaryIO
* Zero-rebuild target VMs
* Reduced migration effort and timeline
* Predictable outcome
* Scalable and repeatable process
* Tight integration with IBM Cloud infrastructure

For more information, see [ConvertIO](https://cloud.ibm.com/catalog?search=primaryio#search_results) {: external}

## Migration Tools
{: #migration-tools}

If you wish to self-migrate, you can still use tools to help with various steps of migration. In addition to services listed above, both Wanclouds and PrimaryIO offer tools that can help with migration. RackWare RMM migration tool deploys seamlessly into your IBM VPC environment and supports auto-provisioning, backup, and recovery strategies across IBM zones and regions. RackWare also integrates with IBM Cloud Object Storage, Block Storage, and Compute to deliver scalable and efficient data protection and workload orchestration. 

RackWare RMM delivers: 
* Seamless migration of physical, virtual, and legacy systems into IBM Cloud VPC environments
* Flexible DR and failover solutions across IBM Cloud VPC, and Bare Metal 
* Unified hybrid cloud control with cost-optimized scaling and provisioning ​
