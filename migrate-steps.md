---

copyright:
  years: 2025
lastupdated: "2025-12-10"

keywords: VSI migration, Classic to VPC, virtual server migration, VPC architecture, security groups, floating IP

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Step-by-step for setting up your VPC
{: #vpc-creating-steps}

This guide provides step-by-step instructions on setting up your first VPC, configuring it and provisioning your VSI in it.
{: shortdesc}

## Migration steps
{: #migration-steps}

Follow these steps to migrate your Classic VSI to VPC VSI. 

### Step 1: Create a resource group
{: #create-rg}

Using the IBM Cloud CLI:

```bash
ibmcloud resource group-create prod-workload-rg
```

### Step 2: Create a VPC infrastructure
{: #create-vpc-infrastructure}

#### Create a VPC
{: #create-vpc}
{: cli}

Using the IBM Cloud CLI:

```bash
ibmcloud is vpc-create production-vpc --resource-group-name prod-workload-rg --address-prefix-management manual
```

#### Create a VPC in the console
{: #create-vpc-console}
{: ui}

1. Navigate to **VPC Infrastructure** > **VPCs**
2. Click **Create**
3. Enter a name (e.g., `production-vpc`)
4. Select a resource group
5. Choose **Manual** for address prefix management
6. Click **Create**

#### Create address prefixes
{: #create-address-prefixes}

Define the IP address range for your VPC:

```bash
ibmcloud is vpc-address-prefix-create production-vpc us-south-1 prefix-us-south-1 10.240.0.0/16
```

#### Create Access control list (ACL)
{: #create-acls}

For enhanced security, it is advisale to create [non-default ACLs](/docs/vpc?topic=vpc-using-acls&interface=ui#updating-the-default-acl) to allow selective traffic to and from your subnet. This step is optional. The ACL can be attached to the subnet.

#### Create subnets
{: #create-subnets}

Create subnets in each zone where you'll deploy VSIs:

```bash
ibmcloud is subnet-create subnet-1 production-vpc us-south-1 --ipv4-cidr-block 10.240.0.0/24
```

For high availability, create subnets in multiple zones:

```bash
ibmcloud is subnet-create subnet-2 production-vpc us-south-2 --ipv4-cidr-block 10.240.1.0/24
ibmcloud is subnet-create subnet-3 production-vpc us-south-3 --ipv4-cidr-block 10.240.2.0/24
```

#### Create a public gateway (optional)
{: #create-public-gateway}

If your VSIs need outbound internet access without floating IPs:

```bash
ibmcloud is public-gateway-create pgw-us-south-1 production-vpc us-south-1
ibmcloud is subnet-update subnet-1 --public-gateway pgw-us-south-1
```

### Step 2: Configure security groups
{: #configure-security-groups}

Create security groups to control traffic to your VSIs:

#### Create a security group for web servers
{: #create-web-sg}

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

#### Create a security group for SSH access
{: #create-ssh-sg}

```bash
ibmcloud is security-group-create ssh-sg production-vpc --resource-group-name prod-workload-rg

# Allow SSH from specific IP range (replace with your IP)
ibmcloud is security-group-rule-add ssh-sg inbound tcp --port-min 22 --port-max 22 --remote 203.0.113.0/24
```

### Step 3: Prepare for VSI creation
{: #prepare-vsi-creation}

#### Generate or import SSH keys
{: #ssh-keys}

VPC VSIs require SSH keys for authentication:

```bash
# Generate a new SSH key pair
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/vpc_rsa

# Import the public key to IBM Cloud
ibmcloud is key-create vpc-key @~/.ssh/vpc_rsa.pub --resource-group-name prod-workload-rg
```

#### Select a VSI profile
{: #select-profile}

List available profiles and choose one that matches or exceeds your Classic VSI specifications. Refer to [compute considerations](/docs/classic-to-vpc?topic=classic-to-vpc-vpc-compute#locations-profiles) for more details.

#### Choose an operating system image
{: #choose-image}

List available images:

```bash
ibmcloud is images --visibility public
```

Select an image that matches your Classic VSI OS, preferably the latest supported version.

### Step 4: Create the VPC VSI from the CLI
{: #create-vpc-vsi-cli}
{: cli}

Create your VSI with all the components configured:

```bash
ibmcloud is instance-create web-server-1 production-vpc us-south-1 bx2-2x8 subnet-1 \
  --image ibm-ubuntu-22-04-minimal-amd64-1 \
  --keys vpc-key \
  --security-groups web-sg,ssh-sg \
  --resource-group-name prod-workload-rg
```


### Step 4: Create the VPC VSI in the console
{: #create-vpc-vsi-ui}
{: ui}

Create your VSI with all the components configured:

1. Navigate to **VPC Infrastructure** > **Virtual server instances**
2. Click **Create**
3. Configure the following:
   * **Name**: web-server-1
   * **Resource group**: production-vpc
   * **Location**: Select region and zone
   * **Image**: Ubuntu 22.04 (or your preferred OS)
   * **Profile**: bx2-2x8 (or your selected profile)
   * **SSH keys**: Select your imported key
   * **VPC**: production-vpc
   * **Subnet**: subnet-1
   * **Security groups**: web-sg, ssh-sg
4. Click **Create virtual server instance**

### Step 5: Attach block storage (if needed)
{: #attach-storage}

If your Classic VSI has additional block storage, create and attach volumes:

```bash
# Create a block storage volume
ibmcloud is volume-create data-volume-1 general-purpose us-south-1 --capacity 100 --iops 3000 --resource-group-name prod-workload-rg

# Attach the volume to the VSI
ibmcloud is instance-volume-attachment-add data-vol-attachment web-server-1 data-volume-1 --auto-delete true
```

### Step 6: Reserve and associate a floating IP
{: #floating-ip}


If your VSI needs public internet access:

```bash
# Reserve a floating IP
ibmcloud is floating-ip-reserve web-server-1-fip --zone us-south-1 --resource-group-name prod-workload-rg

# Associate it with the VSI's network interface
ibmcloud is floating-ip-update web-server-1-fip --nic primary --in web-server-1
```

### Step 7: Migrate data and applications
{: #migrate-data}

Now that your VPC VSI is running, migrate your data and applications:

#### Option 1: Application redeployment (recommended)
{: #redeploy-application}

1. Connect to your new VPC VSI via SSH
2. Install required software and dependencies
3. Deploy your application using your CI/CD pipeline
4. Restore data from backups or replicate from Classic VSI

#### Option 2: Data migration using rsync
{: #rsync-migration}

For data migration, refer to [storage migration](/docs/classic-to-vpc?topic=classic-to-vpc-storage-migrate)

#### Option 3: Database migration
{: #database-migration}

Use appropriate [database migration](/docs/infrastructure-hub?group=database-migration) approach to migrate your databases.

## Post-migration optimization
{: #post-migration}

### Right-size your VSIs
{: #right-size}

Monitor resource utilization and adjust VSI profiles if needed:

```bash
# Stop the VSI
ibmcloud is instance-stop web-server-1

# Update the profile
ibmcloud is instance-update web-server-1 --profile bx2-4x16

# Start the VSI
ibmcloud is instance-start web-server-1
```


## Additional Resources
{: #additional-resources}

* [Terraform tutorial](/docs/ibm-cloud-provider-for-terraform?topic=ibm-cloud-provider-for-terraform-sample_vpc_config)
