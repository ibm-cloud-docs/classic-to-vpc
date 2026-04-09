---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: discovery, inventory

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Discovery of classic infrastructure
{: #discover-classic-infrastructure}

The first step in assessing your environment for migration is to understand what is provisioned and how it is used. You can view the inventory of all resources in your environment from the {{site.data.keyword.cloud}} dashboard. Having an inventory of classic infrastructure resources and their configurations helps with planning, cost estimation, and mapping to target configurations when you provision equivalent resources in the {{site.data.keyword.vpc_full}} (VPC) environment.
{: shortdesc}

## Template for capturing your classic infrastructure footprint
{: #template-capturing-classic-infrasturcture-footprint}

Many specialized tools are available to capture inventory and plan your migration. You can use a sample [template](media/docs/downloads/classic-to-vpc/Classic%20VSI%20Discovery%20Template.pdf) to get started by providing details of the account, compute, storage, and network configurations with high-level application details.

### Account planning for classic to VPC migration
{: #account-planning}

Document your current account structure, including VRF, cloud service endpoints, and compliance requirements to effectively plan your migration to {{site.data.keyword.vpc_short}}, whether you are already in an [enterprise account](/docs/enterprise-management?topic=enterprise-management-what-is-enterprise) or evaluating one.

### Compute resources
{: #compute-resources}

Follow the instructions in [Discovery of Classic Compute Resources](/docs/classic-to-vpc?topic=classic-to-vpc-discovery-of-classic-compute-resources) to identify and capture the compute configuration of the classic virtual servers in your account.

### Storage resources
{: #storage-resources}

Follow the instructions in [Discovery of Classic Storage Resources](/docs/classic-to-vpc?topic=classic-to-vpc-discovery-of-classic-storage-resources) to identify and capture the information about classic storage volumes of in your account.

### Application information
{: #application-information}

Grouping virtual servers by the application they support helps identify all related virtual servers. All virtual servers in a server group must migrate together to reduce communication across {{site.data.keyword.vpc_short}} and classic environments. When you plan data migration, consider the level of data consistency that is needed by the application to restart correctly. For example, an application that can recover from a crash-consistent backup might not need special considerations. In contrast, an application-consistent backup might require quiescing the application, stopping new writes, and taking a coordinated backup across all the virtual servers that support the application. The latter requires more coordination and planning. It can also use [snapshots for block storage](/docs/BlockStorage?topic=BlockStorage-snapshots) and [snapshots for file storage](/docs/FileStorage?topic=FileStorage-snapshots) capabilities. Finally, record any dependencies that a virtual server instance has on other virtual server instances to support wave planning and help ensure connectivity between VPC and classic resources.

## Next steps
{: #next-steps}

After you collect the details for all classic virtual server instances:

1. Compare profiles: Match classic vCPU, memory, and GPU requirements to {{site.data.keyword.vpc_short}} instance profiles.
2. Plan storage migration: Align classic SAN or local attached storage with {{site.data.keyword.vpc_short}} block or file storage.
3. Evaluate billing options: Determine whether hourly or reserved {{site.data.keyword.vpc_short}} profiles are most cost-effective.
4. Assess software licensing: Verify whether required add-ons and licenses are supported in {{site.data.keyword.vpc_short}}. If not, you can install and license them independently.
