---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-02"

keywords: classic to vpc migration, networking

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migration decisions for compute
{: #vpc-decisions-for-compute}

Key references for {{site.data.keyword.vpc_full}} compute in migration scenarios are included here. You can migrate workloads to an existing {{site.data.keyword.vpc_short}} environment. You can also create a new {{site.data.keyword.vpc_short}} as part of the migration, by using the resource provisioning patterns and considerations that are described to support your transition.
{: shortdesc}

## Locations and profiles
{: #locations-profiles}

Locations are composed of regions (specific geographic areas) and zones (fault-tolerant data centers within a region). Review the [locations](/docs/overview?topic=overview-locations) and [service availability](/docs/overview?topic=overview-services_region).

Review the [profiles](/docs/vpc?topic=vpc-profiles) and their availability. Not all profiles might be available in every region. Use the following {{site.data.keyword.cloud_notm}} CLI commands to retrieve the list of profiles available in a region.

```sh
ibmcloud target -r <your-region>
# instance compute profiles
ibmcloud is instance-profiles

#instance dedicated-host profiles
ibmcloud is dedicated-host-profiles

#instance storage profiles
ibmcloud is volume-profiles
```

The flex profiles are recommended targets when you migrate your workloads to VPC. Before you provision the flex profiles, you can validate that the workloads are cross-compatible with both Intel&reg; and AMD architectures. {{site.data.keyword.cloud_notm}} can allocate compute resources on either architecture based on availability, performance, or capacity. Failure to establish that compatibility can result in unexpected behavior or degraded performance.
{: note}

Common profile mappings:
* Classic 2x8 → VPC bxf-2x8 (2 vCPU, 8 GB RAM)
* Classic 4x16 → VPC bxf-4x16 (4 vCPU, 16 GB RAM)
* Classic 8x32 → VPC bxf-8x32 (8 vCPU, 32 GB RAM)

Flex also introduces nano profiles for smaller workloads.

## Instance storage
{: #instance-storage}

[Instance storage](/docs/vpc?topic=vpc-instance-storage) is close to the compute resources of the virtual server and on a high-speed communication channel that is independent from the network. Unlike classic, the data that is stored on instance storage in VPC is ephemeral. It is important to understand the [lifecycle of instance storage](/docs/vpc?topic=vpc-instance-storage#instance-storage-lifecycle). However, flex profiles do not include instance storage. All other profiles provide options with and without instance storage.

## Provisioning your secure environment
{: #deployable-architectures}

Use deployable architectures to provision and update your resources. Use the [Cloud Foundation for VPC deployable architecture](https://cloud.ibm.com/catalog/architecture/deploy-arch-ibm-slz-vpc-9fc0fa64-27af-4fed-9dce-47b3640ba739-global){: external} to automate your deployment architecture.

## High availability and disaster recovery
{: #ha-dr}

Review the [High availability](/docs/vpc?topic=vpc-ha-dr-vpc) features of {{site.data.keyword.vpc_short}}. Learn more about the management [responsibilities](/docs/vpc?topic=vpc-responsibilities-vpc) that you have when you use {{site.data.keyword.cloud_notm}}. The [resiliency](/docs/resiliency) solution guide is a good starting point to understand the resiliency aspects of {{site.data.keyword.cloud_notm}} and designing resilient solutions on {{site.data.keyword.cloud_notm}}.
