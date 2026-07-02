---

copyright:
  years: 2026
lastupdated: "2026-07-02"

keywords: Citrix VPX, load balancer migration, ALB, application load balancer, VPC load balancer, classic to vpc migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from Citrix VPX on IBM Cloud Classic to VPC Application Load Balancer
{: #citrix-vpx-to-vpc-alb}

Migrate your {{site.data.keyword.vpx_full_reg}} {{site.data.keyword.loadbalancer_short}} from {{site.data.keyword.cloud}} Classic Infrastructure to {{site.data.keyword.vpc_full}} Application Load Balancer (ALB). To complete the migration, re-create your load-balancing configuration in {{site.data.keyword.vpc_short}} by using native cloud capabilities.
{: shortdesc}

## Understanding the migration
{: #understanding-migration}

{{site.data.keyword.vpx_full}} on Classic Infrastructure is a virtual appliance that provides advanced load balancing, Secure Sockets Layer (SSL) offloading, and application delivery capabilities. The {{site.data.keyword.vpc_short}} ALB is a fully managed, cloud-native load-balancing service that provides similar functions with simpler management and tighter integration with {{site.data.keyword.vpc_short}} resources.

### Key differences between Citrix VPX and VPC ALB
{: #key-differences}

| Feature | Citrix VPX (Classic) | VPC Application Load Balancer |
| --------- | --------------------- | ------------------------------- |
| **Management** | Self-managed virtual appliance | Fully managed service |
| **Scaling** | Manual scaling requires extra VPX instances | Automatic scaling built-in |
| **High availability** | Requires High Availability (HA) pair configuration | Built-in HA across zones |
| **SSL/TLS** | Managed on VPX appliance | Managed through {{site.data.keyword.secrets-manager_full_notm}} |
| **Health Checks** | Configurable on VPX | Native health check configuration |
| **Pricing** | License + compute costs | Pay-per-use, no license fees |
| **Integration** | Classic network integration | Native VPC integration |
| **Monitoring** | NetScaler monitoring tools | {{site.data.keyword.mon_full_notm}} integration |
{: caption="Comparison of Citrix VPX and VPC ALB" caption-side="bottom"}

## Benefits of migrating to VPC ALB
{: #migration-benefits}

- **Simplified management**: You do not need to manage virtual appliances, patches, or upgrades.
- **Cost optimization**: You can eliminate license costs and reduce additional operational costs.
- **Native cloud integration**: VPC ALB integrates with {{site.data.keyword.vpc_short}} security groups, subnets, and monitoring.
- **Automatic scaling**: The service includes built-in horizontal scaling without manual intervention.
- **Enhanced security**: The service integrates with {{site.data.keyword.secrets-manager_full_notm}} for certificate management.
- **Multi-zone resilience**: The service automatically distributes traffic across availability zones.

## Pre-migration planning
{: #pre-migration-planning}

### 1. Create an inventory of your Citrix VPX configuration
{: #inventory-vpx}

Document your existing Citrix VPX setup:

- **Virtual servers**: List all configured virtual servers (Virtual IP addresses (VIPs)), ports, and protocols.
- **Server pools**: Document backend server pools and their members.
- **Load balancing methods**: Note the algorithms that are used, such as round-robin and least connections.
- **Health checks**: Document health check configurations, including intervals, timeouts, and protocols.
- **SSL certificates**: Inventory all SSL/Transport Layer Security (TLS) certificates and their expiration dates.
- **SSL policies**: Document SSL/TLS versions, cipher suites, and security policies.
- **Persistence settings**: Note session persistence methods, such as source IP and cookie-based persistence.
- **Content switching**: Document any content-based routing rules.
- **Rate limiting**: Note any rate limiting or DDoS protection configurations.
- **Monitoring and logging**: Document the current monitoring and logging setup.

### 2. Understand VPC ALB capabilities
{: #understand-alb}

VPC Application Load Balancer provides the following features:

- **Layer 7 load balancing**: HTTP and HTTPS traffic distribution.
- **SSL/TLS termination**: Offload SSL processing from backend servers.
- **Health checks**: HTTP, HTTPS, and Transmission Control Protocol (TCP) health monitoring.
- **Session persistence**: Cookie-based session affinity.
- **Load-balancing algorithms**: Round robin, weighted round robin, and least connections.
- **Multi-zone deployment**: Automatic distribution across availability zones.
- **Security integration**: Integration with security groups and network Access Control Lists (ACLs).
- **Monitoring**: Integration with {{site.data.keyword.mon_full_notm}} and logging services.

For detailed information, see [About {{site.data.keyword.vpc_short}} Application Load Balancer](/docs/vpc?topic=vpc-load-balancers).

### 3. Map Citrix VPX features to VPC ALB
{: #map-features}

| Citrix VPX Feature | VPC ALB Equivalent | Notes |
| ------------------- | ------------------- | ------- |
| Virtual Server (Virtual IP address (VIP)) | Load Balancer Frontend | Maps 1:1 for basic configurations |
| Service Group | Backend Pool | Contains target instances |
| Server | Pool Member | Individual Virtual Server Instance in the pool |
| Load Balancer (LB) Method (Round Robin) | Round Robin Algorithm | Direct mapping |
| LB Method (Least Connection) | Least Connections Algorithm | Direct mapping |
| LB Method (Weighted) | Weighted Round Robin | Supported by member weights |
| HTTP Monitor | HTTP Health Check | Configure path, interval, time out |
| HTTPS Monitor | HTTPS Health Check | Configure path, interval, time out |
| TCP Monitor | TCP Health Check | Port-based health checking |
| SSL Certificate | Secrets Manager Certificate | Import to Secrets Manager first |
| Cookie Persistence | HTTP Cookie Persistence | Session affinity configuration |
| Source IP Persistence | Source IP Persistence | Available for TCP/User Datagram Protocol (UDP) listeners |
| Content Switching | Multiple Listeners + Policies | Use listener rules for routing |
| SSL offloading | SSL/TLS Termination | Native support with certificate |
{: caption="Feature mapping from Citrix VPX to VPC ALB" caption-side="bottom"}

### 4. Identify migration approach
{: #migration-approach}

Choose the migration strategy that fits your environment:

Parallel deployment (recommended)
:   Deploy VPC ALB alongside the existing Citrix VPX, test thoroughly, and then cut over. This approach minimizes risk and you can roll back.

Phased migration
:   Migrate applications one at a time and gradually move traffic from VPX to ALB. This approach is suitable for complex environments.

Direct cutover
:   Replace VPX with ALB in a single maintenance window. This approach is suitable for simpler configurations with good testing.

## Prerequisites
{: #prerequisites}

Before you start the migration, you can ensure that the following resources are available:

- An existing {{site.data.keyword.vpc_short}} environment, or a plan to create one.
- Backend virtual server instances migrated to {{site.data.keyword.vpc_short}}. For more information, see [Migrating from Classic Virtual Instances to VPC Virtual Instances](/docs/classic-to-vpc?topic=classic-to-vpc-migrate-classic-to-vpc).
- SSL/TLS certificates that are exported from Citrix VPX.
- An instance of {{site.data.keyword.secrets-manager_full_notm}} for certificate management.
- Appropriate Identity and Access Management (IAM) permissions for {{site.data.keyword.vpc_short}} and load balancer management.
- Network connectivity between Classic and {{site.data.keyword.vpc_short}}, if you run a parallel deployment.

For general prerequisites, see [Prerequisites](/docs/classic-to-vpc?topic=classic-to-vpc-key-migration-prerequisites).

## Migration steps
{: #migration-steps}

### Step 1: Export Citrix VPX configuration
{: #export-vpx-config}

Export your current {{site.data.keyword.vpx_full}} configuration for reference.

1. Log in to your {{site.data.keyword.vpx_full}} management interface.
2. Go to **System** > **Diagnostics**.
3. Run the following commands in the command-line interface (CLI) to export the configuration:

   ```bash
   show ns runningConfig
   show lb vserver
   show service
   show serviceGroup
   show lb monitor
   show ssl certKey
   ```
   {: pre}

4. Save the output to use when you configure the VPC ALB.
5. Export the SSL certificates and private keys:

   ```bash
   show ssl certKey <cert-name>
   ```
   {: pre}

### Step 2: Prepare SSL certificates
{: #prepare-certificates}

Import your SSL certificates into {{site.data.keyword.secrets-manager_full_notm}}.

1. Export certificates from Citrix VPX in Privacy-Enhanced Mail (PEM) format.
2. Create an instance of {{site.data.keyword.secrets-manager_short}} if you do not already have one.
3. Import the certificates into {{site.data.keyword.secrets-manager_short}}:

   ```bash
   ibmcloud secrets-manager secret-create \
     --secret-type imported_cert \
     --name "my-app-certificate" \
     --certificate @certificate.pem \
     --private-key @private-key.pem \
     --intermediate @intermediate.pem
   ```
   {: pre}

For more information, see [Managing certificates in {{site.data.keyword.secrets-manager_short}}](/docs/secrets-manager?topic=secrets-manager-certificates).

### Step 3: Create VPC Application Load Balancer
{: #create-alb}

Create the {{site.data.keyword.vpc_short}} Application Load Balancer.

#### Using the IBM Cloud console
{: #create-alb-console}

1. Go to **VPC infrastructure** > **Load balancers**.
2. Click **Create**.
3. Configure the load balancer:
   - **Name**: Enter a descriptive name.
   - **Virtual private cloud**: Select your {{site.data.keyword.vpc_short}}.
   - **Type**: Select **Application Load Balancer**.
   - **Subnets**: Select subnets in multiple zones for high availability.
   - **Resource group**: Select the appropriate resource group.

#### Using the IBM Cloud CLI
{: #create-alb-cli}

```bash
ibmcloud is load-balancer-create my-alb \
  --type application \
  --subnet <subnet-id-zone-1> \
  --subnet <subnet-id-zone-2> \
  --resource-group-name default
```
{: pre}

### Step 4: Configure backend pools
{: #configure-pools}

Create backend pools that correspond to your Citrix VPX service groups.

#### Using the IBM Cloud console
{: #configure-pools-console}

1. In your load balancer details, go to **Back-end pools**.
2. Click **Create**.
3. Configure the pool:
   - **Name**: Enter a descriptive name, such as `web-servers-pool`.
   - **Protocol**: Select HTTP or HTTPS.
   - **Method**: Choose a load-balancing algorithm.
   - **Session stickiness**: Configure if needed.
   - **Health check**: Configure health monitoring.

4. Add pool members:
   - Click **Attach** to add VPC virtual server instances.
   - Select target VPC virtual server instances.
   - Specify the port and weight.

#### Using the IBM Cloud CLI
{: #configure-pools-cli}

```bash
# Create backend pool
ibmcloud is load-balancer-pool-create web-pool <load-balancer-id> \
  --algorithm round_robin \
  --protocol http \
  --health-monitor-delay 5 \
  --health-monitor-max-retries 2 \
  --health-monitor-timeout 2 \
  --health-monitor-type http \
  --health-monitor-url-path /health

# Add pool members
ibmcloud is load-balancer-pool-member-create <load-balancer-id> web-pool \
  --target <vsi-id> \
  --port 80 \
  --weight 50
```
{: pre}

### Step 5: Configure health checks
{: #configure-health-checks}

Citrix VPX monitors map to VPC ALB health checks as follows:

| Health Check Parameter | Description | Recommended Value |
| ---------------------- | ------------- | ------------------- |
| **Protocol** | HTTP, HTTPS, or TCP | Match your application |
| **Port** | Backend server port | Same as pool member port |
| **URL Path** | Health check endpoint | Health or status |
| **Interval** | Time between checks | 5-10 seconds |
| **Timeout** | Response timeout | 2-5 seconds |
| **Max Retries** | Failed checks before unhealthy | 2-3 retries |
{: caption="Health check configuration parameters" caption-side="bottom"}

Example health check configuration:

```bash
ibmcloud is load-balancer-pool-update <load-balancer-id> <pool-id> \
  --health-monitor-delay 5 \
  --health-monitor-max-retries 2 \
  --health-monitor-timeout 2 \
  --health-monitor-type http \
  --health-monitor-url-path /health
```
{: pre}

### Step 6: Configure front end listeners
{: #configure-listeners}

Create listeners that correspond to your Citrix VPX virtual servers:

#### HTTP listener
{: #http-listener}

```bash
ibmcloud is load-balancer-listener-create <load-balancer-id> \
  --port 80 \
  --protocol http \
  --default-pool <pool-id>
```
{: pre}

#### HTTPS listener with SSL termination
{: #https-listener}

```bash
ibmcloud is load-balancer-listener-create <load-balancer-id> \
  --port 443 \
  --protocol https \
  --certificate-instance <secrets-manager-certificate-crn> \
  --default-pool <pool-id>
```
{: pre}

#### Configure listener policies (for content switching)
{: #listener-policies}

If you use Citrix VPX content switching, create listener policies:

```bash
# Create policy for path-based routing
ibmcloud is load-balancer-listener-policy-create <load-balancer-id> <listener-id> \
  --action forward \
  --priority 1 \
  --name api-routing \
  --target <api-pool-id>

# Add rule to match path
ibmcloud is load-balancer-listener-policy-rule-create <load-balancer-id> <listener-id> <policy-id> \
  --condition contains \
  --type path \
  --value /api
```
{: pre}

### Step 7: Configure session persistence
{: #configure-persistence}

Configure session affinity to match your Citrix VPX persistence settings.

HTTP cookie-based persistence

```bash
ibmcloud is load-balancer-pool-update <load-balancer-id> <pool-id> \
  --session-persistence-type http_cookie
```
{: pre}

Source IP persistence for TCP or User Datagram Protocol (UDP) traffic

```bash
ibmcloud is load-balancer-pool-update <load-balancer-id> <pool-id> \
  --session-persistence-type source_ip
```
{: pre}

### Step 8: Configure security
{: #configure-security}

Secure your VPC ALB with security groups.

1. Create a security group for the load balancer:

   ```bash
   ibmcloud is security-group-create alb-security-group <vpc-id>
   ```
   {: pre}

2. Add inbound rules for client traffic:

   ```bash
   # Allow HTTP
   ibmcloud is security-group-rule-add <sg-id> inbound tcp \
     --port-min 80 --port-max 80

   # Allow HTTPS
   ibmcloud is security-group-rule-add <sg-id> inbound tcp \
     --port-min 443 --port-max 443
   ```
   {: pre}

3. Update backend virtual server instance security groups to allow traffic from the ALB subnets.

### Step 9: Test the configuration
{: #test-configuration}

Thoroughly test the VPC ALB before cutover:

1. **Domain Name System (DNS) testing**: Create a test DNS entry that points to the ALB hostname.
2. **Functional testing**: Verify that all application features work correctly.
3. **Load testing**: Test with production-like traffic volumes.
4. **Failover testing**: Verify health checks and automatic failover.
5. **SSL/TLS testing**: Verify certificate configuration and cipher suites.
6. **Session persistence testing**: Verify that sticky sessions work as expected.
7. **Performance testing**: Compare response times with Citrix VPX.

Test the configuration by using the ALB hostname.

```bash
# Get ALB hostname
ibmcloud is load-balancer <load-balancer-id> --output json | grep hostname
```
{: pre}

### Step 10: Plan and execute cutover
{: #cutover}

Run the migration cutover.

#### Pre-cutover checklist
{: #pre-cutover-checklist}

- [ ] VPC ALB fully configured and tested
- [ ] All SSL certificates that are imported and validated
- [ ] Backend pools are working correctly and responding
- [ ] Monitoring and alerting configured
- [ ] Rollback plan documented
- [ ] Stakeholders notified of maintenance window
- [ ] DNS Time to Live (TTL) reduced (for example, to 300 seconds)

#### Cutover steps
{: #cutover-steps}

1. **Schedule the maintenance window**: Choose a minimal user activity period.
2. **Enable parallel operation**: Run both VPX and ALB at the same time.
3. **Update DNS**: Point the DNS records to the {{site.data.keyword.vpc_short}} ALB hostname.
4. **Monitor traffic**: Watch for errors and performance issues.
5. **Verify functions**: Test the critical application paths.
6. **Monitor for 24 - 48 hours**: Keep Citrix VPX running as a backup.
7. **Decommission VPX**: After you validate the migration, remove Citrix VPX.

#### Rollback procedure
{: #rollback-procedure}

If issues occur, complete the following steps:

1. Update DNS to point back to Citrix VPX.
2. Wait for DNS propagation based on the TTL.
3. Investigate and resolve issues with VPC ALB.
4. Plan another cutover attempt.

## Post-migration optimization
{: #post-migration}

### Monitor and tune performance
{: #monitor-performance}

Configure monitoring for the VPC ALB.

1. **Enable {{site.data.keyword.mon_full_notm}}**:
   ```bash
   ibmcloud is load-balancer-update <load-balancer-id> \
     --logging-datapath-active true
   ```
   {: pre}

2. **Key metrics to monitor**:
   - Active connections
   - Throughput (bytes/s)
   - Response time
   - Health check status
   - Backend pool member health
   - SSL/TLS handshake time

3. **Set up alerts** for:
   - When all backend members are not working well
   - High error rates (`4xx`, `5xx` Hypertext Transfer Protocol (HTTP) status codes)
   - When the limits to the connections are approaching their end
   - When your certificate is about to expire

### Optimize configuration
{: #optimize-config}

Fine-tune the VPC ALB configuration.

- **Adjust health check intervals**: Balance responsiveness and backend load.
- **Review the load-balancing algorithm**: Optimize based on actual traffic patterns.
- **Update pool member weights**: Distribute load based on capacity.
- **Review security group rules**: Help ensure minimal privilege access.
- **Optimize SSL/TLS settings**: Use modern cipher suites and protocols.

### Implement high availability
{: #implement-ha}

Enhance resilience with the following practices:

- **Multi-zone deployment**: Help ensure that subnets span multiple availability zones.
- **Multiple backend pools**: Create redundant pools for critical applications.
- **Cross-region failover**: Consider the Global Load Balancer for disaster recovery.
- **Automated scaling**: Use instance groups with auto-scaling for backend virtual server instances.

### Cost optimization
{: #cost-optimization}

Optimize costs with the following practices:

- **Right-size backend Virtual server instances**: Monitor the usage and adjust profiles.
- **Review data transfer costs**: Optimize for in-region traffic.
- **Consolidate load balancers**: Combine multiple applications where appropriate.
- **Use private load balancers**: Use them for internal-only applications.

## Troubleshooting
{: #troubleshooting}

### Common issues and solutions
{: #common-issues}

Backend pool members show as unhealthy
:   Verify that the security group rules allow traffic from the ALB subnets.
:   Check that the health check configuration matches the backend application.
:   Verify that the backend virtual server instances are running and accessible.
:   Review the health check logs in {{site.data.keyword.mon_full_notm}}.

SSL/TLS certificate errors
:   Verify that the certificate is properly imported into {{site.data.keyword.secrets-manager_short}}.
:   Check the certificate expiration date.
:   Make sure that the certificate chain is complete, including the intermediate certificates.
:   Verify that the certificate matches the domain name.

Session persistence is not working
:   Verify that the session persistence type matches the application requirements.
:   Check whether the backend application overrides cookies.
:   Review the load balancer logs for session routing.
:   Test with different browsers or clients.

Performance is worse than on Citrix VPX
:   Review backend virtual server instance performance and sizing.
:   Check the network latency between the ALB and the backends.
:   Verify that the health check intervals are not too aggressive.
:   Review the load-balancing algorithm selection.
:   Consider enabling connection pooling on the backends.

You cannot access the load balancer
:   Verify that the security group rules allow inbound traffic.
:   Check the network Access Control Lists (ACLs) on the ALB subnets.
:   Verify DNS resolution to the ALB hostname.
:   Review the {{site.data.keyword.vpc_short}} routing tables.

For extra troubleshooting, see [Troubleshooting VPC](/docs/vpc?topic=vpc-troubleshooting-vpc).

## Comparison: Feature parity checklist
{: #feature-parity}

Confirm feature parity after migration by verifying the following items:

- [ ] All virtual servers (Virtual IP addresses (VIPs)) are migrated to listeners.
- [ ] All backend pools are configured with the correct members.
- [ ] Load-balancing algorithms match the original configuration.
- [ ] Health checks are configured and functioning.
- [ ] SSL/TLS certificates are installed and validated.
- [ ] Session persistence works as expected.
- [ ] Content-based routing rules are migrated, if applicable.
- [ ] Monitoring and alerting are configured.
- [ ] Security policies are implemented.
- [ ] Performance meets or exceeds the baseline.
- [ ] Documentation is updated.

## Next steps
{: #next-steps}

- [Set up monitoring and logging](/docs/vpc?topic=vpc-monitoring-metrics-alb)
- [Configure auto-scaling for backend virtual server instances](/docs/vpc?topic=vpc-creating-auto-scale-instance-group)
- [Implement Global Load Balancer for multi-region high availability](/docs/cis?topic=cis-global-load-balancer-glb-concepts)
- [Review VPC security best practices](/docs/vpc?topic=vpc-security-in-your-vpc)

## Additional resources
{: #additional-resources}

- [VPC Application Load Balancer documentation](/docs/vpc?topic=vpc-load-balancers)
- [VPC Application Load Balancer Application Programming Interfaces (API) reference](/apidocs/vpc#list-load-balancers)
- [{{site.data.keyword.secrets-manager_full_notm}} documentation](/docs/secrets-manager)
- [{{site.data.keyword.secrets-manager_full}} documentation](/docs/secrets-manager)
- [IBM Cloud Secrets Manager documentation](/docs/secrets-manager)
- [VPC security groups documentation](/docs/vpc?topic=vpc-using-security-groups)
- [Citrix VPX to cloud-native migration patterns](https://www.ibm.com/think/architectures/patterns){: external}
