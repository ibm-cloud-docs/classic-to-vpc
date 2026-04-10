---

copyright:
  years: 2026, 2026

lastupdated: "2026-04-10"

keywords: virtual server instance migration, Classic to VPC, virtual server migration, VPC architecture, security groups, floating IP

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Setting up your VPC environment
{: #vpc-creating-steps}

The following steps explain how to set up and configure your first {{site.data.keyword.vpc_full}} environment and provision a virtual server within it.
{: shortdesc}

## Step by step migration guide
{: #migration-steps}

Follow the steps to migrate a Classic virtual server to {{site.data.keyword.vpc_short}} virtual server.

### Creating a resource group
{: #creating-rg}
{: #step-1}

Create a resource group to organize and manage related {{site.data.keyword.cloud}} resources, such as your virtual server instances, networking components, and storage.

Using the {{site.data.keyword.cloud}} CLI, you can create a resource group by specifying a meaningful name that reflects its purpose. If you already have an appropriate resource group, you can reuse it instead of creating a new one.

1. Log in to the {{site.data.keyword.cloud}} CLI.
2. Create a resource group by running the following command:

```bash
ibmcloud resource group-create prod-workload-rg
```

### Creating a VPC infrastructure
{: #creating-vpc-infrastructure}
{: #step-2}

This step explains how to create the infrastructure required for your {{site.data.keyword.vpc_short}} environment.

#### Creating a VPC environment
{: #creating-vpc-environment}
{: cli}

Create {{site.data.keyword.vpc_full}} to establish a logically isolated network environment for your cloud resources. The VPC serves as the foundation for networking components such as subnets, gateways, and virtual server instances.

Using the {{site.data.keyword.cloud}} CLI, create a VPC by specifying the VPC name, resource group, and address prefix management option.

Run the following command to create a VPC with manual address prefix management:

```bash
ibmcloud is vpc-create production-vpc --resource-group-name prod-workload-rg --address-prefix-management manual
```

#### Creating a VPC in the console
{: #creating-vpc-console}
{: ui}

Follow the steps to create {{site.data.keyword.vpc_full}} in the console:

1. From the web console, go to **VPC Infrastructure** > **VPCs** and click **Create**
2. Enter a name (for example, `production-vpc`)
3. Select a resource group
4. Choose **Manual** for address prefix management and click **Create**

#### Create address prefixes
{: #create-address-prefixes}

Run the following command to define the IP address range that is used by your VPC.

```bash
ibmcloud is vpc-address-prefix-create production-vpc us-south-1 prefix-us-south-1 10.240.0.0/16
```

#### Creating access control list (ACL)
{: #create-acls}

For enhanced security, it is advisable to create [nondefault ACLs](/docs/vpc?topic=vpc-using-acls&interface=ui#updating-the-default-acl) to allow selective traffic to and from your subnet. This step is optional. The ACL can be attached to the subnet.

#### Creating subnets
{: #creating-subnets}

Create subnets to define IP address ranges within your VPC where virtual server instances are deployed. Each subnet is associated with a specific zone and provides network connectivity for the resources that are present in that zone.

Run the following command to create subnets in each zone where you deploy virtual servers:

```bash
ibmcloud is subnet-create subnet-1 production-vpc us-south-1 --ipv4-cidr-block 10.240.0.0/24
```

For high availability, create subnets in multiple zones:

```bash
ibmcloud is subnet-create subnet-2 production-vpc us-south-2 --ipv4-cidr-block 10.240.1.0/24
ibmcloud is subnet-create subnet-3 production-vpc us-south-3 --ipv4-cidr-block 10.240.2.0/24
```

#### Creating a public gateway (optional)
{: #creating-public-gateway}

Run the following command if your virtual server instances need outbound internet access without floating IPs:

```bash
ibmcloud is public-gateway-create pgw-us-south-1 production-vpc us-south-1
ibmcloud is subnet-update subnet-1 --public-gateway pgw-us-south-1
```

### Configuring security groups
{: #configure-security-groups}
{: #step-3}

Create security groups to define and control inbound and outbound network traffic for your virtual server instances. Security groups act as virtual firewalls that specify rules based on protocols, ports, source, and destination IP addresses. You can create multiple security groups to apply different sets of rules to different virtual server instances.

#### Creating a security group for web servers
{: #creating-web-sg}

Use a security group to define the network access rules for your web servers.Run the following command to create a security group for web servers:

```bash
ibmcloud is security-group-create web-sg production-vpc --resource-group-name prod-workload-rg
```

Add rules to allow HTTP and HTTPS traffic:

```bash
# Allow inbound HTTP
ibmcloud is security-group-rule-add web-sg inbound tcp --port-min 80 --port-max 80

# Allow inbound HTTPS
ibmcloud is security-group-rule-add web-sg inbound tcp --port-min 443 --port-max 443

# Allow all outbound traffic (default)
ibmcloud is security-group-rule-add web-sg outbound all
```

#### Creating a security group for SSH access
{: #creating-ssh-sg}

Create a security group to allow SSH access to your virtual server instances. Run the following command to create a security group for SSH access:

```bash
ibmcloud is security-group-create ssh-sg production-vpc --resource-group-name prod-workload-rg

# Allow SSH from specific IP range (replace with your IP)
ibmcloud is security-group-rule-add ssh-sg inbound tcp --port-min 22 --port-max 22 --remote 203.0.113.0/24
```

### Preparing for virtual server instance creation
{: #preparing-virtual server instance-creation}
{: #step-4}

Before you create a virtual server instance, complete the required preparation steps for a successful deployment.

#### Generating or importing SSH keys
{: #ssh-keys}

VPC virtual server instances require SSH keys for authentication: Run the following command to generate a new SSH key pair and import the public key to {{site.data.keyword.cloud}}. You can also import an existing public key if you have one.

```bash
# Generate a new SSH key pair
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/vpc_rsa

# Import the public key to IBM Cloud
ibmcloud is key-create vpc-key @~/.ssh/vpc_rsa.pub --resource-group-name prod-workload-rg
```

#### Selecting a virtual server instance profile
{: #select-profile}

Make a list of the available profiles, and select one that meets or exceeds your Classic virtual server instance specifications. Refer to [compute considerations](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-decisions-for-compute#locations-profiles) for more details.

#### Choosing an operating system image
{: #choose-image}

Run the following command to list available images:

```bash
ibmcloud is images --visibility public
```

Select an image that matches your Classic virtual server instance OS, preferably the most recent supported version.

### Step 4: Create the VPC virtual server instance from the CLI
{: #create-vpc-virtual server instance-cli}
{: cli}

Create your virtual server instance by configuring all required components.

```bash
ibmcloud is instance-create web-server-1 production-vpc us-south-1 bx2-2x8 subnet-1 \
  --image ibm-ubuntu-22-04-minimal-amd64-1 \
  --keys vpc-key \
  --security-groups web-sg,ssh-sg \
  --resource-group-name prod-workload-rg
```

### Creating the VPC virtual server instance in the console
{: #create-vpc-virtual server instance-ui}
{: ui}
{: #step-5}

Create your virtual server instance by configuring all required components.

1. From the web console, go to **VPC Infrastructure** > **Virtual server instances** and click **Create**
2. Configure the following attributes:
   * **Name**: web-server-1
   * **Resource group**: production-vpc
   * **Location**: Select region and zone
   * **Image**: Ubuntu 22.04 (or your preferred OS)
   * **Profile**: bx2-2x8 (or your selected profile)
   * **SSH keys**: Select your imported key
   * **VPC**: production-vpc
   * **Subnet**: subnet-1
   * **Security groups**: web-sg, ssh-sg
3. Click **Create a virtual server instance**

### Attaching block storage (if needed)
{: #attach-block-storage}
{: #step-6}

Run the following command if your Classic virtual server instance has more block storage, create and attach volumes:

```bash
# Create a block storage volume
ibmcloud is volume-create data-volume-1 general-purpose us-south-1 --capacity 100 --iops 3000 --resource-group-name prod-workload-rg

# Attach the volume to the virtual server instance
ibmcloud is instance-volume-attachment-add data-vol-attachment web-server-1 data-volume-1 --auto-delete true
```

### Reserving and associating a floating IP
{: #reserving-and-associating-a-floating-ip}
{: #step-7}

Run the following command if your virtual server instance needs public internet access:

```bash
# Reserve a floating IP
ibmcloud is floating-ip-reserve web-server-1-fip --zone us-south-1 --resource-group-name prod-workload-rg

# Associate it with the virtual server instance's network interface
ibmcloud is floating-ip-update web-server-1-fip --nic primary --in web-server-1
```

### Migrating data and applications
{: #migrate-data-and-applications}
{: #step-8}

Now that your VPC virtual server instance is running, migrate your data and applications by using one of the following options based on your specific requirements and constraints.

#### Option 1: Application redeployment (recommended)
{: #redeploy-application}

1. Connect to your new VPC virtual server instance through SSH
2. Install required software and dependencies
3. Deploy your application by using your CI/CD pipeline
4. Restore data from backups or replicate from Classic virtual server instance

#### Option 2: Data migration that uses rsync
{: #rsync-migration}

For data migration, refer to [storage migration](/docs/classic-to-vpc?topic=classic-to-vpc-data-migration-classic-to-vpc)

#### Option 3: Database migration
{: #database-migration}

Use appropriate [database migration](/docs/infrastructure-hub?group=database-migration) approach to migrate your databases.

## Post-migration optimization
{: #post-migration-optimization}

After migration, monitor the performance and resource usage of your VPC virtual server instance. Optimize the configuration as needed to meet your workload requirements while you minimize costs.

### Right-size your virtual server instances
{: #right-size-instances}

Right‑size your virtual server instances by adjusting CPU, memory, and storage to match actual workload requirements. Right sizing helps improve performance and avoids unnecessary resource costs.
Monitor resource usage and adjust virtual server instance profiles if needed

```bash
# Stop the virtual server instance
ibmcloud is instance-stop web-server-1

# Update the profile
ibmcloud is instance-update web-server-1 --profile bx2-4x16

# Start the virtual server instance
ibmcloud is instance-start web-server-1
```

## Additional resources
{: #additional-resources}

The following resources offer more guidance and supplemental information.

* [Terraform tutorial](/docs/ibm-cloud-provider-for-terraform?topic=ibm-cloud-provider-for-terraform-sample_vpc_config)
