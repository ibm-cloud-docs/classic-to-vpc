---

copyright:
  years: 2025
lastupdated: "2025-11-07"

keywords: discovery, inventory

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}


# Discovery
{: #discovery}

The first step in assessing your environmemt for migration is to understand what is provisioned and how is it being used. The inventory of all resources in your environment can be viewed from the IBM Cloud dashboard. Having a ready reckoner of the classic infrastructure resources and their configuration helps in planning, cost estimation and mapping to target configurations when you provision equivalent resources in the VPC environment. 
{: shortdesc}


## Template
{: #template}

There are many purpose built tools to capture inventory and plan your migration. We provide a sample [template](https://cloud.ibm.com/media/docs/downloads/classic-to-vpc/Classic%20VSI%20Discovery%20Template.pdf) to get you started. This captures details of the account, compute, storage and network configuration as well as high-level application details.

### Account Information
{: #account-information}

Capture the account id, VRF status, total number of resources etc.

### Compute Resources 
{: #compute-resources}

Follow the instructions captured in compute [section](/docs/classic-to-vpc?topic=classic-to-vpc-discovery-of-classic-compute-resources) to identify and capture the compute configuration of the classic virtual servers in your account.

### Storage Resources
{: #storage-resources}

Follow the instructions captured in storage [section](/docs/classic-to-vpc?topic=classic-to-vpc-discovery-of-classic-storage-resources) to identify and capture the information about classic storage volumes of in your account.



### Application Information
{: #application-information}

Grouping virtual servers by the application they are supporting allows you to identify all related virtual servers. All virtual servers in a server group should be migrated together to reduce communication across VPC and classic environments. When you plan for data migration, it is important to consider the level of data consistency needed by the application to restart correctly. For instance, an application which can recover from a crash consistent backup might not need any special considerations while an application consistent backup might require that you quiesce the application, stop processing new writes and take a co-ordinated backup across all the virtual servers supporting the application. The latter requires more co-ordination and planning, it can also leverage [snapshots for block storage](/docs/BlockStorage?topic=BlockStorage-snapshots) and [snapshots for file storage](/docs/FileStorage?topic=FileStorage-snapshots) capabilities. Finally, record any dependencies which a VSI has on other VSIs to perform wave planning and connectivity between VPC and classic resources.


## Next Steps
{: #next-steps}

Once you’ve collected the details for all Classic VSIs:

1. **Compare Profiles**  
   Match Classic vCPU, memory, and GPU requirements to VPC instance profiles.  

2. **Plan Storage Migration**  
   Align Classic SAN or local attached storage with VPC block storage or file storage.  

3. **Evaluate Billing Options**  
   Decide whether hourly or reserved profiles in VPC are most cost-effective.  

4. **Assess Software Licensing**  
   Check whether the required add-ons and licenses are supported in VPC. If not, you can install and license them yourself.
