---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: virtual server instance migration, Classic to VPC, virtual server migration, VPC architecture, security groups, floating IP

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from Classic virtual server instance to VPC virtual server instance
{: #migrate-classic-to-vpc}

The migration process moves virtual server instances from Classic infrastructure to {{site.data.keyword.vpc_full}} (VPC) by re-creating the infrastructure in {{site.data.keyword.vpc_short}} with equivalent or enhanced capabilities. This migration guide provides a comprehensive overview of the migration process, including planning, execution, and post-migration optimization.
{: shortdesc}

## Understanding the migration process
{: #migration-process-explaination}

Migrating from Classic virtual server instance to {{site.data.keyword.vsi_is_full}} involves creating a basic VPC{{site.data.keyword.vpc_short}} landing zone and migrating over your virtual server instances from Classic to {{site.data.keyword.vpc_short}}.
With this approach you can:

* Implement modern security practices with security groups
* Take advantage of VPC's enhanced networking capabilities
* Benefit from improved performance and scalability

## VPC architecture overview
{: #vpc-architecture-overview}

Before you start the migration process, it's important to understand the {{site.data.keyword.vpc_short}} architecture and how it differs from Classic infrastructure.

### Sample VPC architecture diagram
{: #sample-vpc-diagram}

The following diagram illustrates a typical {{site.data.keyword.vpc_short}} setup with a single virtual server instance, including all essential components.

![Diagram depicting a simple VPC environment with virtual server instance](../images/VPC.drawio.svg "Diagram depicting a simple VPC environment with a virtual server instance"){: caption="Diagram depicting a simple VPC environment with a virtual server instance" caption-side="bottom"}

### Key VPC components
{: #key-vpc-components}

The following list of {{site.data.keyword.vpc_short}}components and their descriptions help you understand the {{site.data.keyword.vpc_short}} architecture. Understanding these {{site.data.keyword.vpc_short}} components is essential for a successful migration

- **VPC**: An isolated network environment where you deploy your resources. Each {{site.data.keyword.vpc_short}} has its own IP address range (CIDR block).

- **Subnets**: Subdivisions of your {{site.data.keyword.vpc_short}}'s IP address range. Subnets are zone-specific and contain your virtual server instances.

- **Virtual server instance**: The compute resource is equivalent to a Classic virtual server instance. {{site.data.keyword.vpc_short}} virtual server instances offer more flexible profiles and better performance.

- **Security groups**: Stateful firewalls that control inbound and outbound traffic at the virtual server instance level.

- **Floating IP**: A public IP address can be associated with a virtual server instance to enable internet access. Similar to Classic public IPs, it can be easily reassigned to other virtual server instances and uses network address translation (NAT) to route traffic to the instance’s private IP address.

- **Public gateway**: Enables virtual server instances with only private IPs to access the internet for outbound connections (for example, software updates).

- **Network ACLs (Access Control Lists)**: Stateless firewalls that operate at the subnet level, providing an extra layer of security.

- **Block storage**: Persistent storage that can be attached to virtual server instances.

## Pre-migration planning
{: #pre-migration-planning}

The migration process can begin when you complete the prerequisites and planned your migration strategy. The following steps outline the key activities to prepare for a successful migration.

### Inventory of your Classic infrastructure
{: #inventory-of-classic-infrastructure}
{: #step-1}

Document your existing Classic virtual server instance configuration:

Go through [Discovery](/docs/classic-to-vpc?topic=classic-to-vpc-discovery) for all the different infrastructure and application aspects that include:

* Virtual server instance specifications (CPU, RAM, storage)
* Operating system and version
* Network configuration (public or private IPs, VLANs)
* Security groups and firewall rules
* Block storage volumes and their sizes
* Backup schedules and retention policies
* Application dependencies and integrations


### Mapping Classic resources to VPC equivalents
{: #map-resources}
{: #step-2}

| Classic Resource | VPC Equivalent | Notes |
| ----------------- | ---------------- | ------- |
| Virtual server instance Profile | VPC Profile | Choose an equivalent or better profile (for example, B1.2x8 → bxf-2x8) |
| Public IP | Floating IP | Must be explicitly associated with virtual server instance |
| Private IP | Private IP | Automatically assigned from subnet range |
| VLAN | Subnet | Subnets are zone-specific in VPC |
| Security Group | Security Group | VPC security groups are more flexible |
| Block Storage | Block Storage Volume | VPC offers better IOPS performance |
| Bare Metal | Dedicated Host or virtual server instance | Consider dedicated hosts for isolation |
{: caption="Classic to VPC resource mapping" caption-side="bottom"}

Refer to [compute considerations](/docs/classic-to-vpc?topic=classic-to-vpc-compute_migrate) for various options and considerations.

### Designing your VPC architecture
{: #design-vpc}
{: #step-3}

Plan your {{site.data.keyword.vpc_short}} structure:

1. **Choose a region and zones**: Select regions close to your users. Use multiple zones for high availability.
2. **Define IP address ranges**: Choose nonoverlapping CIDR blocks (for example, 10.240.0.0/16 for {{site.data.keyword.vpc_short}}, 10.240.0.0/24 for subnets).
3. **Plan subnet layout**: Create subnets for different tiers (web, app, database) or environments (dev, staging, prod).
4. **Design security groups**: Define rules based on the principle of least privilege.
5. **Plan connectivity**: Determine whether you need {{site.data.keyword.vpn_vpc_full}}, {{site.data.keyword.dl_short}}, or {{site.data.keyword.tg_short}} for hybrid connectivity.

### Verifying prerequisites
{: #verifying-prerequisites}
{: #step-4}

Confirm that you completed all prerequisites that are listed in [Prerequisites](/docs/classic-to-vpc?topic=classic-to-vpc-key-migration-prerequisites):

* VRF-enabled account
* IBM Cloud CLI with {{site.data.keyword.vpc_short}} plug-in installed
* Appropriate IAM permissions
* Sufficient quota for {{site.data.keyword.vpc_short}} resources

### Step 1: Create VPC Infrastructure
{: #create-vpc-infra}

- **Create a VPC** in an appropriate resource group and assign a nonoverlapping CIDR for address range.

- **Create nondefault ACLs (optional)** to secure your inbound and outbound network traffic.

- **Create subnets** in each zone by carving out appropriate subnets from the address range.

- **Create a public gateway (optional)** for outbound internet access without floating IPs.

### Step 2: Configure security groups
{: #configure-security-groups}

Based on the applications that are running on your virtual server instances, create security groups to allow inbound traffic (for example, SSH/HTTPS).

Allowing all outbound traffic is usually set by default and often required for successful provisioning of virtual server instances.
{: note}

### Step 3: Create the VPC virtual server instance
{: #create-vpc-virtual server instance}

Create {{site.data.keyword.vpc_short}} virtual server by selecting an appropriate profile that matches or exceeds your classic virtual server instance resources. Also, create a matching operating system image that is at the most recent supported version. Confirm that you have an SSH key available.

### Step 4: Attach storage
{: #attach-storage}

You can optionally create and attach secondary volumes of both block storage or file share types. Prepare the volumes by creating partitions and file systems to receive the data from your classic instance.

### Step 5: Reserve and associate a floating IP
{: #reserve-associate-floating-ip}

Assign a floating IP to your virtual server instance if you need inbound public internet access.

### Step 7: Migrate data and applications
{: #migrate-data}

Now that your {{site.data.keyword.vpc_short}} virtual server instance is running, migrate your data and applications. Rebuild your application and all its dependencies by running your continuous deployment (CD) pipeline. Migrate your data by using backup and restore or application-specific strategies (for example, database migration). Refer to [storage migration](/docs/classic-to-vpc?topic=classic-to-vpc-storage-migrate) for details.

### Step 8: Update DNS and networking
{: #update-dns}

Update your DNS records to point to the new {{site.data.keyword.vpc_short}} virtual server instance:

1. Note the floating IP address of your {{site.data.keyword.vpc_short}} virtual server instance
2. Update your DNS A records to point to the new IP
3. Consider using a reduced TTL during migration for quick rollback
4. Update any firewall rules or security policies

### Step 9: Test and validate
{: #test-validate}

Thoroughly test your migrated application:

1. **Functional testing**: Verify all application features work correctly
2. **Performance testing**: Compare performance with Classic virtual server instance
3. **Security testing**: Verify security groups and network access
4. **Backup testing**: Confirm backup and restore procedures work
5. **Monitoring**: Set up monitoring and alerting for the new virtual server instance



### Step 10: Cutover and decommission
{: #cutover}

After you complete testing and confirm that the migration is successful, follow these steps:

1. Schedule a maintenance window
2. Conduct final data synchronization
3. Update DNS to point to VPC virtual server instance
4. Monitor for issues
5. Keep the Classic virtual server running for a rollback period (for example, 1-2 weeks)
6. Decommission Classic virtual server instance after successful cutover

## Post-migration optimization
{: #post-migration}

After migration, optimize your VPC environment:

### Right-size your virtual server instances
{: #right-size}

Monitor resource usage and adjust virtual server instance profiles as needed. Changing a profile requires explicitly stopping and starting the virtual server instance.

### Implement high availability
{: #implement-ha}

Consider these HA options to improve resilience and uptime:

* Deploy virtual server instances across multiple zones
* Use load balancers to distribute traffic
* Implement auto-scaling for dynamic workloads
* Set up backup and disaster recovery

### Optimize costs
{: #optimize-costs}

Use the following strategies to optimize costs in your virtual server environment:

* Use reserved capacity for predictable workloads
* Implement auto-scaling to match demand
* Review and remove unused resources
* Consider using instance groups for similar virtual server instances

### Enhance security
{: #enhance-security}

Apply the following practices to strengthen the security of your virtual server environment:

* Regularly review and update security group rules
* Implement network ACLs for extra protection
* Enable VPC flow logs for network monitoring
* Use IBM Cloud Security and Compliance Center


## Next actions
{: #next-actions}

Complete the following steps to finalize your migration and prepare for ongoing operations:

* Validate your migration
* Finalize and cutover
* [Set up monitoring and logging](/docs/vpc?topic=vpc-ibm-monitoring)
* [Implement backup strategies](/docs/vpc?topic=vpc-backups-vpc-best-practices)

## Related links
{: #Related-links}

* [VPC documentation](/docs/vpc)
* [VPC CLI reference](/docs/vpc?topic=vpc-vpc-reference)
* [VPC API reference](/apidocs/vpc)
* [VPC Terraform provider](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs)
* [IBM Cloud Architecture Center](https://www.ibm.com/architectures/hybrid)
