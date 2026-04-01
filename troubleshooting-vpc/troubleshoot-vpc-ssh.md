---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-01"

keywords: question about classic to vpc migration, accessing my vpc, ssh

subcollection: classic-to-vpc

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# Why am I unable to connect to my {{site.data.keyword.vsi_is_short}} using SSH
{: #troubleshoot-vpc-ssh-issues}
{: troubleshoot}

You provision a new VSI, and it shows as running, but you cannot connect to it over SSH.
{: shortdesc}

## What's happening
{: #tsSymptoms-vpc-ssh}

Your SSH session times out while you connect to the host, or authentication fails after connection.

## Why it’s happening
{: #tsCauses-vpc-ssh}

- The security group does not allow SSH traffic.
- A floating IP address not associated with the VSI.
- The SSH key is not properly configured.

## How to fix it
{: #tsResolve-vpc-ssh}

1. Review the security groups and verify that SSH traffic is allowed (port 22). For more information, see [Security Groups](/docs/vpc?group=security-groups).
2. Associate a floating IP address with the VSI when you use a public IP address. For more information, see [Creating network interfaces with floating IP addresses](/docs/vpc?topic=vpc-fip-working).
3. Reconfigure the SSH and reprovision the instance if necessary. For more information, see [SSH Keys](/docs/vpc?group=ssh-keys).
