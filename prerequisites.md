---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: pre-requisites, classic-to-vpc, tools

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Prerequisites
{: #key-migration-prerequisites}

Before you begin the Classic to {{site.data.keyword.vpc_full}} migration process, confirm that the required tools, capabilities, and configurations are in place as you prepare for the migration. The information on this page describes the key prerequisites that are needed to successfully complete the migration steps.
{: shortdesc}

## Account virtual routing function enablement
{: #vrf-enablement}

To connect over {{site.data.keyword.IBM}} private network to {{site.data.keyword.vpc_short}} resources, your account with classic infrastructure resources needs to have virtual routing function ([VRF](/docs/account?topic=account-vrf-service-endpoint&interface=ui)) enabled. Confirm that your account is VRF enabled. No additional steps are required if VRF is already enabled.

Before you enable VRF, read the [FAQ](/docs/account?topic=account-vrf-faqs) to understand and plan for enablement. A short intermittent connectivity loss can occur between your existing classic servers on the private network during the migration process.

## {{site.data.keyword.cloud_notm}} CLI
{: #ibm-cloud-cli}

The {{site.data.keyword.cloud}} Command Line Interface (CLI) provides commands for managing resources in {{site.data.keyword.cloud_notm}}. When you install the stand-alone [IBM Cloud CLI](/docs/cli?topic=cli-getting-started), you get only the CLI itself without any recommended plug-ins or tools. Install the necessary plug-ins to work with your environment.

You can find more information about plug-ins and command help in the [CLI reference](/docs/cli?topic=cli-ibmcloud_cli) section of the IBM Cloud CLI documentation.

## Quota increases
{: #quota-increases}

Before you provision your target environment on VPC, review the [Quota and Service Limits for VPC](/docs/vpc?topic=vpc-quotas). To request a limit increase, open a [Support Case](/unifiedsupport/cases/form).

In some migration scenarios, you might need to increase your quota on the classic source environment. For example, if you are at your maximum storage quota and need to add a temporary disk during migration, you must request a quota increase first. For more information, see [Managing storage limits](/docs/BlockStorage?topic=BlockStorage-managingstoragelimits).

## Support and help
{: #support-and-help}

For support or questions, see [Getting help and support](/docs/support?topic=support-using-avatar).

## See also
{: #see-also-prerequisites}

* [Discovery of classic infrastructure](/docs/classic-to-vpc?topic=classic-to-vpc-discover-classic-infrastructure). If you have not yet inventoried your classic environment, complete discovery before you start provisioning VPC resources.
* [Setting up your VPC environment](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-creating-steps). Follow these step-by-step instructions to create a VPC, subnets, security groups, and a virtual server instance.
* [Migration decisions for compute](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-decisions-for-compute). Review this topic for profile selection, high availability options, and deployable architectures to consider before provisioning.
* [Migrating from Classic virtual server instance to VPC virtual server instance](/docs/classic-to-vpc?topic=classic-to-vpc-migrate-classic-to-vpc). Follow this guide to complete the end-to-end migration process after prerequisites are confirmed.
