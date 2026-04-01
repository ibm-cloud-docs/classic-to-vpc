---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-01"

keywords: question about classic to vpc migration, performance

subcollection: classic-to-vpc

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# Why is my application performance slower than Classic VSI?
{: #troubleshoot-vpc-performance-issues}
{: troubleshoot}

Your application is running slowly and is not handling requests at the same scale as your classic VSI.
{: shortdesc}

## What’s happening
{: #tsSymptoms-vpc-performance}

Your application is running slowly or taking longer to handle requests. CPU utilization or IO wait times might be high.

## Why it’s happening
{: #tsCauses-vpc-performance}

- The VSI profile is undersized.
- Block storage IOPS or bandwidth is insufficient.
- Network bandwidth is constrained.

## How to fix it
{: #tsResolve-vpc-performance}

1. Upgrade to a larger VSI profile.
2. Increase block storage [IOPS](/docs/vpc?topic=vpc-adjusting-volume-iops) or [bandwidth](/docs/vpc?topic=vpc-adjusting-volume-throughput).
3. Use a profile with higher network bandwidth.
