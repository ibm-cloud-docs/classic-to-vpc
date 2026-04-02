---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-02"

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

## IBM Cloud CLI
{: #ibm-cloud-cli}

The {{site.data.keyword.cloud.notm}} Command Line Interface (CLI) provides commands for managing resources in {{site.data.keyword.cloud_notm}}. When you install the stand-alone [IBM Cloud CLI](/docs/cli?topic=cli-getting-started), you get only the CLI itself without any recommended plug-ins or tools. Install the necessary plug-ins to work with your environment.

You can find more information about plug-ins and command help in the [CLI reference](/docs/cli?topic=cli-ibmcloud_cli) section of the {{site.data.keyword.cloud.notm}} CLI documentation.

## Support and help
{: #support-and-help}

For support assistance or questions, refer to the [Support Center](/docs/account?topic=account-using-avatar&interface=ui).

### Quota increases
{: #quota-increases}

To validate your quota needs before deploying your target environment on VPC, refer to the [Quota and Service Limits for VPC](/docs/vpc?topic=vpc-quotas). To request a limit increase, open a [Support Case](https://cloud.ibm.com/unifiedsupport/cases/form){: external}

On the classic source environment, you might need to increase the quota in some situations to support migration. One scenario might be when adding a new temporary disk when you are at maximum storage quota. For more information, see [Managing storage limits](/docs/BlockStorage?topic=BlockStorage-managingstoragelimits).
