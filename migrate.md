---

copyright:
  years: 2025
lastupdated: "2025-12-10"

keywords: VSI migration, Classic to VPC, virtual server migration, VPC architecture, security groups, floating IP

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from Classic VSI to VPC VSI
{: #migrate-classic-to-vpc}

This guide provides a process to migrate your Virtual Server Instances (VSIs) from Classic Infrastructure to {{site.data.keyword.vpc_full}} (VPC). The migration involves recreating your infrastructure in VPC with equivalent or enhanced capabilities.
{: shortdesc}

## Understanding the migration process
{: #migration-process-overview}

Migrating from Classic VSI to {{site.data.keyword.vsi_is_full}} (VPC VSI) involves creating a basic VPC landing zone and migrating over your VSI(s) from Classic to VPC. This approach allows you to:

* Implement modern security practices with security groups
* Take advantage of VPC's enhanced networking capabilities
* Benefit from improved performance and scalability

## VPC architecture overview
{: #vpc-architecture}

Before starting your migration, it's important to understand the VPC architecture and how it differs from Classic Infrastructure.

### Sample VPC architecture diagram
{: #sample-vpc-diagram}

The following diagram illustrates a typical VPC setup with a single VSI, including all essential components:

![Diagram depicting a simple VPC environment with VSI](images/VPC.drawio.svg "Diagram depicting a simple VPC environment with a VSI"){: caption="Diagram depicting a simple VPC environment with a VSI" caption-side="bottom"}

### Key VPC components
{: #vpc-components}

Understanding these VPC components is essential for a successful migration:

**{{site.data.keyword.vpc_full}}**
:   An isolated network environment where you deploy your resources. Each VPC has its own IP address range (CIDR block).

**Subnets**
:   Subdivisions of your VPC's IP address range. Subnets are zone-specific and contain your VSIs.

**{{site.data.keyword.vsi_is_short}}**
:   The compute resource equivalent to Classic VSI. VPC VSIs offer more flexible profiles and better performance.

**Security Groups**
:   Stateful firewalls that control inbound and outbound traffic at the VSI level.

**Floating IP**
:   A public IP address that can be associated with a VSI to provide internet access. Similar to Classic public IPs but can be easily reassigned to other VSIs. Uses network address translation (NAT) to route traffic to the VSI's private IP address.

**Public Gateway**
:   Enables VSIs with only private IPs to access the internet for outbound connections (e.g., software updates).

**Network ACLs (Access Control Lists)**
:   Stateless firewalls that operate at the subnet level, providing an additional layer of security.

**{{site.data.keyword.block_storage_is_short}}**
:   Persistent storage that can be attached to VSIs.

## Pre-migration planning
{: #pre-migration-planning}

Before beginning the migration, complete these planning steps:

### 1. Inventory your Classic infrastructure
{: #inventory-classic}

Document your existing Classic VSI configuration:

Go through [Discovery](/docs/classic-to-vpc?topic=classic-to-vpc-discovery) for all the different infrastructure and application aspects. This will include

* VSI specifications (CPU, RAM, storage)
* Operating system and version
* Network configuration (public/private IPs, VLANs)
* Security groups and firewall rules
* Block storage volumes and their sizes
* Backup schedules and retention policies
* Application dependencies and integrations


### 2. Map Classic resources to VPC equivalents
{: #map-resources}

| Classic Resource | VPC Equivalent | Notes |
|-----------------|----------------|-------|
| VSI Profile | VPC Profile | Choose equivalent or better profile (e.g., B1.2x8 → bxf-2x8) |
| Public IP | Floating IP | Must be explicitly associated with VSI |
| Private IP | Private IP | Automatically assigned from subnet range |
| VLAN | Subnet | Subnets are zone-specific in VPC |
| Security Group | Security Group | VPC security groups are more flexible |
| Block Storage | Block Storage Volume | VPC offers better IOPS performance |
| Bare Metal | Dedicated Host or VSI | Consider dedicated hosts for isolation |
{: caption="Classic to VPC resource mapping" caption-side="bottom"}

Refer to [compute considerations](/docs/classic-to-vpc?topic=classic-to-vpc-compute_migrate) for various options and considerations.

### 3. Design your VPC architecture
{: #design-vpc}

Plan your VPC structure:

1. **Choose a region and zones**: Select regions close to your users. Use multiple zones for high availability.
2. **Define IP address ranges**: Choose non-overlapping CIDR blocks (e.g., 10.240.0.0/16 for VPC, 10.240.0.0/24 for subnets).
3. **Plan subnet layout**: Create subnets for different tiers (web, app, database) or environments (dev, staging, prod).
4. **Design security groups**: Define rules based on the principle of least privilege.
5. **Plan connectivity**: Determine if you need {{site.data.keyword.vpn_vpc_full}}, {{site.data.keyword.dl_short}}, or {{site.data.keyword.tg_short}} for hybrid connectivity.

### 4. Verify prerequisites
{: #verify-prerequisites}

Ensure you have completed all prerequisites listed in [Pre-requisites](/docs/classic-to-vpc?topic=classic-to-vpc-pre-requisites):

* VRF-enabled account
* IBM Cloud CLI with VPC plugin installed
* Appropriate IAM permissions
* Sufficient quota for VPC resources

### Step 1: Create VPC Infrastructure
{: #create-vpc-infra}

**Create a VPC** in an appropriate resource group and that you assign a non-overlapping CIDRs for address range.

**Create non-default ACLs (optional)** to secure your inbound and outbound network traffic.

**Create subnets** in each zone by carving out appropriate subnets from the address range.

**Create a public gateway (optional)** for outbound internet access without floating IPs.

### Step 2: Configure security groups
{: #configure-security-groups}

Based on the applications running on your VSIs, create security groups to allow inbound traffic (e.g., SSH/HTTPS). 

Note: Allowing all outbound traffic is usually set by default and often required for sucessful provisioning of VSIs.
{: note}

### Step 3: Create the VPC VSI
{: #create-vpc-vsi}

Create a VPC VSI by selecting an appropriate profile that matches or exceeds your classic VSI resources and a matching operating system image that is at latest supported version. Ensure that you have SSH key available.

### Step 4: Attach storage
{: #attach-storage}

You can optionally create and attach secondary volumes of both block storage or file share types. Prepare the volumes by creating partitions and filesystems to receive the data from your classic instance.

### Step 5: Reserve and associate a floating IP
{: #floating-ip}

Assign a floating IP to your VSI if you need inbound public internet access.

### Step 7: Migrate data and applications
{: #migrate-data}

Now that your VPC VSI is running, migrate your data and applications. Rebuild your application and all its dependencies by running your continuous deployment (CD) pipeline. Migrate your data using backup and restore or application specific strategies (e.g., database migration). Refer to [storage migration](/docs/classic-to-vpc?topic=classic-to-vpc-storage-migrate) for details.

### Step 8: Update DNS and networking
{: #update-dns}

Update your DNS records to point to the new VPC VSI:

1. Note the floating IP address of your VPC VSI
2. Update your DNS A records to point to the new IP
3. Consider using a low TTL during migration for quick rollback
4. Update any firewall rules or security policies

### Step 9: Test and validate
{: #test-validate}

Thoroughly test your migrated application:

1. **Functional testing**: Verify all application features work correctly
2. **Performance testing**: Compare performance with Classic VSI
3. **Security testing**: Verify security groups and network access
4. **Backup testing**: Ensure backup and restore procedures work
5. **Monitoring**: Set up monitoring and alerting for the new VSI



### Step 10: Cutover and decommission
{: #cutover}

Once testing is complete and you're confident in the migration:

1. Schedule a maintenance window
2. Perform final data synchronization
3. Update DNS to point to VPC VSI
4. Monitor for issues
5. Keep Classic VSI running for a rollback period (e.g., 1-2 weeks)
6. Decommission Classic VSI after successful cutover

## Post-migration optimization
{: #post-migration}

After migration, optimize your VPC environment:

### Right-size your VSIs
{: #right-size}

Monitor resource utilization and adjust VSI profiles if needed. This requires your to explicitly stop and start your VSI.

### Implement high availability
{: #implement-ha}

Consider these HA options:

* Deploy VSIs across multiple zones
* Use load balancers to distribute traffic
* Implement auto-scaling for dynamic workloads
* Set up backup and disaster recovery

### Optimize costs
{: #optimize-costs}

* Use reserved capacity for predictable workloads
* Implement auto-scaling to match demand
* Review and remove unused resources
* Consider using instance groups for similar VSIs

### Enhance security
{: #enhance-security}

* Regularly review and update security group rules
* Implement network ACLs for additional protection
* Enable VPC flow logs for network monitoring
* Use IBM Cloud Security and Compliance Center


## Next steps
{: #next-steps}

* Validate your migration
* Finalize and cutover
* [Set up monitoring and logging](/docs/vpc?topic=vpc-ibm-monitoring)
* [Implement backup strategies](/docs/vpc?topic=vpc-backups-vpc-best-practices)

## Additional resources
{: #additional-resources}

* [VPC documentation](/docs/vpc)
* [VPC CLI reference](/docs/vpc?topic=vpc-vpc-reference)
* [VPC API reference](/apidocs/vpc)
* [VPC Terraform provider](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs)
* [IBM Cloud Architecture Center](https://www.ibm.com/architectures/hybrid)
