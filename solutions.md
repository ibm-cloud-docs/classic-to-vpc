---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: migration solutions, classic to vpc migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migration Solutions
{: #solutions}

Depending on your environment and workloads, you can choose from one of the various solutions to migrate to {{site.data.keyword.vpc_full}}.

The approach 'one-size-fits-all' does not apply to migration. The choice of solutions and tools can be guided by the complexity of your existing classic environment and your modernization goals.
{: shortdesc}

## A typical migration process
{: #typical-process}

Migration is a multi-step process, typically involving but not limited to following steps.

### Discovering classic resources
{: #discovering-classic-resources}

   Discover your existing classic resources that include compute, storage, and network resources. This step involves creating a comprehensive inventory of workloads, configurations, dependencies, and usage patterns across your current environment. A thorough discovery phase establishes a clear baseline, helping you assess migration readiness, reduce risks, and plan an efficient transition to the target {{site.data.keyword.vpc_short}} platform.

### Mapping to VPC
{: #mapping-to-vpc}

   Map your existing environment to {{site.data.keyword.vpc_short}} landing zone, including [locations](/docs/overview?topic=overview-locations), VPCs, network, storage profiles, server profiles and OS versions.This step aligns with the current workloads and configurations with the target architecture and deployment standards. Proper mapping helps identify gaps, compatibility considerations, and required changes, enabling a smoother, well-governed migration to the VPC-based environment.

### Provision your landing zone
{: #provision-landing-zone}

   Provision your base {{site.data.keyword.vpc_short}} environment that includes subnets, virtual server instances, and empty storage volumes. You can use [deployable architectures](/docs/secure-enterprise?topic=secure-enterprise-understand-module-da) that include deployable architecture solutions, such as the [VPC landing zone](https://cloud.ibm.com/catalog/architecture/deploy-arch-ibm-slz-vpc){: external}.

### Deploying your application
{: #deploy-application}

   Deploy your application and its dependencies onto the {{site.data.keyword.vpc_short}} virtual server instance by using your continuous delivery pipeline.This step automates the build, test, and deployment processes to provide consistent, repeatable, and reliable application delivery. Leveraging a continuous delivery pipeline helps reduce deployment errors, accelerate release cycles, and maintain alignment with DevOps and operational best practices in the target environment.

### Migrating your data
{: #migrating-your-data}

   Migrate your application data by copying files and folders from the source data volumes to the target environment. Everything stays in sync and your application has access to the same data after it moves. If needed, you can also set up a regular, scheduled sync to automatically transfer any new or changed data until you are ready for the final cutover.

### Validating your application
{: #validating-your-application}

   Validate your application in the target environment to make sure everything is working as expected after the migration. This step includes checking that the application starts correctly, data is accessible, integrations are functioning, and performance meets requirements. Testing this stage can catch any issues early, build confidence in the new setup, and support a smooth transition before moving fully to production.

### Cutting over from Classic to VPC
{: #Cutting-over-from-classic-to-vpc}

   Stop all write operations on the source system and switch your production traffic from the classic environment to {{site.data.keyword.vpc_short}}. This cutover step helps ensure that all users and applications start using the new environment without any data changes happening in the old environment. After you have confirmed a successful migration, plan to safely deprovision and clean up the old resources to avoid unnecessary costs and reduce ongoing maintenance.

The various sections in the guide provide guidance, pointers, and existing tools and techniques to go through the migration process in a 'do it yourself' (DIY) manner.
{: note}

## Using third-party services for migration
{: #third-party-migration-services}

If you prefer to use external third-party services and capabilities for migration, refer to the following options that you can evaluate.

### VPC+ Cloud Migration
{: #vpc-cloud-migration}

If you want to migrate your {{site.data.keyword.cloud}} classic infrastructure (compute, network, and storage) to {{site.data.keyword.vpc_short}}, you can use {{site.data.keyword.vpc-plus-migration}}. {{site.data.keyword.vpc-plus-migration}} is a third-party, software-based, migration-as-a-service solution, provided by Wanclouds, for migrating components from classic infrastructure to your {{site.data.keyword.vpc_short}}. With {{site.data.keyword.vpc-plus-migration}}, you can discover and choose resources for migration, create, and set up those resources in your {{site.data.keyword.vpc_short}} environment. You can also run and manage your {{site.data.keyword.vpc_short}} environment from within the tool.

You can migrate the following key elements of {{site.data.keyword.cloud_notm}} classic infrastructure to {{site.data.keyword.vpc_short}}:

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

For more information, see [Getting started with VPC+ migration](/docs/wanclouds-vpc-plus?topic=wanclouds-vpc-plus-getting-started-tutorial).

### ConvertIO Workload Migration
{: #convertio-workload-migration}

PrimaryIO empowers organizations to move to the cloud seamlessly. You can adopt PrimaryIO as a primary production environment or use it for disaster recovery. By offering a comprehensive suite of tools and services, PrimaryIO facilitates efficient, secure, and low-risk migration of VMware workloads to {{site.data.keyword.cloud_notm}} help ensure minimal disruption and maximum operational continuity.

Benefits of using PrimaryIO for {{site.data.keyword.cloud_notm}} migration:

* Complete VM migration - managed by PrimaryIO
* Zero-rebuild target VMs
* Reduced migration effort and timeline
* Predictable outcome
* Scalable and repeatable process
* Tight integration with {{site.data.keyword.cloud_notm}} infrastructure

For more information, see [ConvertIO](https://cloud.ibm.com/catalog?search=primaryio#search_results) {: external}

### Migration tools
{: #migration-tools}

If you want to self-migrate, you can still use tools to help with various steps of migration. In addition to the services listed in the other sections, both Wanclouds and PrimaryIO offer tools that can help with migration. RackWare RMM migration tool deploys seamlessly into your IBM {{site.data.keyword.vpc_short}} environment and supports auto-provisioning, backup, and recovery strategies across IBM zones and regions. RackWare also integrates with {{site.data.keyword.cos_full}}, Block Storage, and Compute to deliver scalable and efficient data protection and workload orchestration.

What RackWare RMM delivers:
* Seamless migration of physical, virtual, and legacy systems into {{site.data.keyword.vpc_short}} environments
* Flexible DR and failover solutions across {{site.data.keyword.vpc_short}}, and {{site.data.keyword.bm_is_full}}
* Unified hybrid cloud control with cost-optimized scaling and provisioning
