---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: question about classic to vpc migration, internet access

subcollection: classic-to-vpc

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# Why my application can not access the internet?
{: #troubleshoot-vpc-internet-access-issues}
{: troubleshoot}

Your application that is running on {{site.data.keyword.vsi_is_short}} cannot connect to public internet sites.
{: shortdesc}

## What is happening?
{: #tsSymptoms-vpc-internet-access}

Your application fails to establish outbound connections to the public internet. DNS resolution is successful, but the connection fails.

## Why it is happening?
{: #tsCauses-vpc-internet-access}

- A floating IP address or public gateway is not configured.
- The security group blocks outbound traffic.
- The network ACL blocks traffic.

## How do you fix it?
{: #tsResolve-vpc-internet-access}

1. [Associate a floating IP address](/docs/vpc?topic=vpc-vni-add-fip) for inbound and outbound access.
2. Attach a public gateway to the subnet for outbound-only access.
3. Verify that the security group allows outbound traffic.
4. Check network ACL rules, if configured.
