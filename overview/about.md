---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: migration, migrate, migrating, migrate infrastructure, cloud migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# About migration
{: #about-migration-infra}

Your decision to migrate can be driven by many factors, such as modernization, cost savings, consolidation, or closing a data center. You can also migrate to better align applications with cloud environments or to adopt new technologies like {{site.data.keyword.vpc_full}}. No matter the reason, a migration can range from something as simple as migrating a single virtual server instance to something much more complex, such as migrating an entire application environment, pod, or full data center along with all its supporting components.
{: shortdesc}

The four-phase migration approach: Assess, Plan, Migrate, and Validate, applies to all infrastructure migration scenarios on {{site.data.keyword.cloud_notm}}. For a full description of each phase, see [About migration](/docs/infrastructure-hub?topic=infrastructure-hub-about-migration-infra) in the Infrastructure Hub documentation.

## Planning your Classic to VPC migration
{: #planning-classic-to-vpc}

If you are migrating from classic infrastructure to VPC, the following topics guide you through the Assess, Plan, and Migrate phases in sequence.

### Discover your classic infrastructure
{: #discover-classic-infra-links}

Before you plan capacity or select target profiles, build a complete inventory of what you are migrating.

* [Discovery of classic infrastructure](/docs/classic-to-vpc?topic=classic-to-vpc-discover-classic-infrastructure). Read this topic for an overview of discovery, account planning, and application grouping.
* [Discovery of classic compute resources](/docs/classic-to-vpc?topic=classic-to-vpc-discover-classic-compute-resources). Use this topic to build a per-virtual-server inventory of vCPU, memory, OS, storage, and network configuration.
* [Discovery of classic storage resources](/docs/classic-to-vpc?topic=classic-to-vpc-discover-classic-storage-resources). Use this topic to identify and size block, file, portable, and local storage.

### Meet prerequisites
{: #meet-prerequisites-links}

Complete these setup steps before you provision any VPC resources.

* [Prerequisites for migration](/docs/classic-to-vpc?topic=classic-to-vpc-key-migration-prerequisites). Complete this topic to enable VRF, set up the IBM Cloud CLI, and request VPC quota increases.

### Set up your VPC environment and migrate
{: #setup-and-migrate-links}

After discovery and prerequisites are complete, provision your target environment and move your workloads.

* [Setting up your VPC environment](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-creating-steps). Follow these step-by-step instructions to provision resource groups, a VPC, subnets, security groups, SSH keys, and virtual server instances.
* [Migration decisions for compute](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-decisions-for-compute). Review this topic for profile selection, high availability and disaster recovery options, and deployable architectures.
* [Migrating from Classic virtual server instance to VPC virtual server instance](/docs/classic-to-vpc?topic=classic-to-vpc-migrate-classic-to-vpc). Follow this guide to complete the end-to-end migration process, from pre-migration planning through cutover and post-migration optimization.

## Next steps
{: #next-steps}

* To compare migration tools (VPC+ Cloud Migration, RackWare RMM, and DIY automation) across Classic-to-Classic, Classic-to-VPC, and on-premises-to-VPC scenarios, see [Migration solutions](/docs/infrastructure-hub?topic=infrastructure-hub-about-migration-infra#migration-solutions) in the Infrastructure Hub documentation.
* For Classic-to-VPC-specific tools, including ConvertIO (PrimaryIO) for VMware workloads, see [Classic-to-VPC migration solutions](/docs/classic-to-vpc?topic=classic-to-vpc-solutions) in this guide.
* Contact your IBM Cloud Customer Success Manager (CSM) or IBM Cloud Seller for planning, migration assistance, and other queries. If you don't have an assigned CSM or IBM Cloud Seller, IBM reaches out to the primary account contact through email.
