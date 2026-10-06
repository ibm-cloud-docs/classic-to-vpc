---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: migration solutions, classic to vpc migration, third-party migration tools

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migration solutions
{: #solutions}

Depending on your environment and workloads, you can choose from several solutions to migrate to {{site.data.keyword.vpc_full}}. There is no one-size-fits-all approach. The right choice depends on the complexity of your classic environment and your modernization goals.
{: shortdesc}

For a side-by-side comparison of VPC+ Cloud Migration, RackWare RMM, and DIY automation, see [Migration solutions](/docs/infrastructure-hub?topic=infrastructure-hub-about-migration-infra#migration-solutions) in the Infrastructure Hub documentation.

## VPC+ Cloud Migration
{: #vpc-cloud-migration}

{{site.data.keyword.vpc-plus-migration}} is a third-party, software-based migration-as-a-service solution, provided by Wanclouds, for migrating components from {{site.data.keyword.cloud_notm}} classic infrastructure to {{site.data.keyword.vpc_short}}. The tool discovers your classic resources, creates equivalent resources in {{site.data.keyword.vpc_short}}, and lets you manage your VPC environment from within the tool.

You can migrate the following classic infrastructure elements by using {{site.data.keyword.vpc-plus-migration}}:

* Subnets
* Virtual server instances
* Dedicated hosts
* Storage volumes (primary and secondary)
* Security groups
* Load balancers
* Firewall (ACL) configuration
* VPN configuration
* SSH keys
* Public gateways



For full feature details, supported migration motions, and a getting-started guide, see [Getting started with VPC+ Cloud Migration](/docs/wanclouds-vpc-plus?topic=wanclouds-vpc-plus-getting-started-tutorial).

## ConvertIO Workload Migration (PrimaryIO)
{: #convertio-workload-migration}

ConvertIO, provided by PrimaryIO, is a migration tool that focuses on VMware workload migration to {{site.data.keyword.cloud_notm}}. It is suited for organizations that want a managed, low-disruption path to move VM-based workloads to {{site.data.keyword.vpc_short}} without manually rebuilding target instances.

Key benefits of ConvertIO for {{site.data.keyword.cloud_notm}} migration:

* Complete VM migration managed by PrimaryIO
* Zero-rebuild target VMs
* Reduced migration effort and timeline
* Predictable outcome
* Scalable and repeatable process
* Tight integration with {{site.data.keyword.cloud_notm}} infrastructure

For more information, see [ConvertIO in the IBM Cloud catalog](https://cloud.ibm.com/catalog?search=primaryio#search_results){: external}.

This tool is specific to Classic-to-VPC VMware workload migrations and is not included in the Infrastructure Hub cross-scenario comparison table.
{: note}

## RackWare RMM
{: #rackware-rmm}

RackWare Management Module (RMM) is a third-party automated migration solution that supports Classic-to-Classic, Classic-to-VPC, on-premises-to-VPC, and cloud-to-VPC migration motions.

For full details on supported migration motions, architecture diagrams, limitations, and step-by-step guides, see [RackWare RMM](/docs/infrastructure-hub?topic=infrastructure-hub-about-migration-infra#rackware-migration) in the Infrastructure Hub documentation.

## DIY migration
{: #diy-migration}

If you prefer to manage your migration without a third-party tool, the rest of this guide covers a do-it-yourself (DIY) approach. The DIY sections provide guidance on discovery, planning, provisioning, data migration, and cutover by using {{site.data.keyword.cloud_notm}} native capabilities and open tooling.

For a comparison of DIY automation against VPC+ and RMM across all migration scenarios, see [Migration solutions](/docs/infrastructure-hub?topic=infrastructure-hub-about-migration-infra#migration-solutions) in the Infrastructure Hub documentation.
