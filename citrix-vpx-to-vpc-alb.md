---

copyright:
  years: 2025
lastupdated: "2026-04-17"

keywords: Citrix VPX, load balancer migration, ALB, application load balancer, VPC load balancer, classic to vpc migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from Citrix VPX on IBM Cloud Classic to VPC Application Load Balancer
{: #citrix-vpx-to-vpc-alb}

This guide provides a comprehensive process to migrate your Citrix NetScaler VPX load balancer from IBM Cloud Classic Infrastructure to {{site.data.keyword.vpc_full}} Application Load Balancer (ALB). The migration involves recreating your load balancing configuration in VPC with native cloud capabilities.
{: shortdesc}

## Understanding the migration
{: #understanding-migration}

Citrix NetScaler VPX on Classic Infrastructure is a virtual appliance that provides advanced load balancing, SSL offloading, and application delivery capabilities. {{site.data.keyword.vpc_short}} Application Load Balancer is a fully managed, cloud-native load balancing service that provides similar functionality with simplified management and better integration with VPC resources.

### Key differences between Citrix VPX and VPC ALB
{: #key-differences}

| Feature | Citrix VPX (Classic) | VPC Application Load Balancer |
|---------|---------------------|-------------------------------|
| **Management** | Self-managed virtual appliance | Fully managed service |
| **Scaling** | Manual scaling, requires additional VPX instances | Automatic scaling built-in |
| **High Availability** | Requires HA pair configuration | Built-in HA across zones |
| **SSL/TLS** | Managed on VPX appliance | Managed through IBM Cloud Secrets Manager |
| **Health Checks** | Configurable on VPX | Native health check configuration |
| **Pricing** | License + compute costs | Pay-per-use, no license fees |
| **Integration** | Classic network integration | Native VPC integration |
| **Monitoring** | NetScaler monitoring tools | IBM Cloud Monitoring integration |
{: caption="Comparison of Citrix VPX and VPC ALB" caption-side="bottom"}

## Benefits of migrating to VPC ALB
{: #migration-benefits}

* **Simplified management**: No need to manage virtual appliances, patches, or upgrades
* **Cost optimization**: Eliminate license costs and reduce operational overhead
* **Native cloud integration**: Seamless integration with VPC security groups, subnets, and monitoring
* **Automatic scaling**: Built-in horizontal scaling without manual intervention
* **Enhanced security**: Integration with IBM Cloud Secrets Manager for certificate management
* **Multi-zone resilience**: Automatic distribution across availability zones

## Pre-migration planning
{: #pre-migration-planning}

### 1. Inventory your Citrix VPX configuration
{: #inventory-vpx}

Document your existing Citrix VPX setup:

* **Virtual servers**: List all configured virtual servers (VIPs), ports, and protocols
* **Server pools**: Document backend server pools and their members
* **Load balancing methods**: Note the algorithms used (round-robin, least connections, etc.)
* **Health checks**: Document health check configurations (intervals, timeouts, protocols)
* **SSL certificates**: Inventory all SSL/TLS certificates and their expiration dates
* **SSL policies**: Document SSL/TLS versions, cipher suites, and security policies
* **Persistence settings**: Note session persistence methods (source IP, cookie-based, etc.)
* **Content switching**: Document any content-based routing rules
* **Rate limiting**: Note any rate limiting or DDoS protection configurations
* **Monitoring and logging**: Document current monitoring and logging setup

### 2. Understand VPC ALB capabilities
{: #understand-alb}

Review VPC Application Load Balancer features:

* **Layer 7 load balancing**: HTTP/HTTPS traffic distribution
* **SSL/TLS termination**: Offload SSL processing from backend servers
* **Health checks**: HTTP, HTTPS, and TCP health monitoring
* **Session persistence**: Cookie-based session affinity
* **Load balancing algorithms**: Round-robin, weighted round-robin, least connections
* **Multi-zone deployment**: Automatic distribution across availability zones
* **Security integration**: Works with security groups and network ACLs
* **Monitoring**: Integration with IBM Cloud Monitoring and logging services

For detailed information, see [About {{site.data.keyword.vpc_short}} Application Load Balancer](/docs/vpc?topic=vpc-load-balancers).

### 3. Map Citrix VPX features to VPC ALB
{: #map-features}

| Citrix VPX Feature | VPC ALB Equivalent | Notes |
|-------------------|-------------------|-------|
| Virtual Server (VIP) | Load Balancer Frontend | Maps 1:1 for basic configurations |
| Service Group | Backend Pool | Contains target instances |
| Server | Pool Member | Individual VSI in the pool |
| LB Method (Round Robin) | Round Robin Algorithm | Direct mapping |
| LB Method (Least Connection) | Least Connections Algorithm | Direct mapping |
| LB Method (Weighted) | Weighted Round Robin | Supported with member weights |
| HTTP Monitor | HTTP Health Check | Configure path, interval, timeout |
| HTTPS Monitor | HTTPS Health Check | Configure path, interval, timeout |
| TCP Monitor | TCP Health Check | Port-based health checking |
| SSL Certificate | Secrets Manager Certificate | Import to Secrets Manager first |
| Cookie Persistence | HTTP Cookie Persistence | Session affinity configuration |
| Source IP Persistence | Source IP Persistence | Available for TCP/UDP listeners |
| Content Switching | Multiple Listeners + Policies | Use listener rules for routing |
| SSL Offloading | SSL/TLS Termination | Native support with certificate |
{: caption="Feature mapping from Citrix VPX to VPC ALB" caption-side="bottom"}

### 4. Identify migration approach
{: #migration-approach}

Choose the appropriate migration strategy:

**Parallel deployment (Recommended)**
:   Deploy VPC ALB alongside existing Citrix VPX, test thoroughly, then cutover. Minimizes risk and allows rollback.

**Phased migration**
:   Migrate applications one at a time, gradually moving traffic from VPX to ALB. Suitable for complex environments.

**Direct cutover**
:   Replace VPX with ALB in a single maintenance window. Suitable for simpler configurations with good testing.

## Prerequisites
{: #prerequisites}

Before starting the migration, ensure you have:

* An existing VPC environment or plan to create one
* Backend VSIs migrated to VPC (see [Migrating from Classic VSI to VPC VSI](/docs/classic-to-vpc?topic=classic-to-vpc-migrate-classic-to-vpc))
* SSL/TLS certificates exported from Citrix VPX
* IBM Cloud Secrets Manager instance for certificate management
* Appropriate IAM permissions for VPC and load balancer management
* Network connectivity between Classic and VPC (if running parallel deployment)

For general prerequisites, see [Pre-requisites](/docs/classic-to-vpc?topic=classic-to-vpc-key-migration-prerequisites).

## Migration steps
{: #migration-steps}

### Step 1: Export Citrix VPX configuration
{: #export-vpx-config}

Export your current Citrix VPX configuration for reference:

1. Log in to your Citrix VPX management interface
2. Navigate to **System** > **Diagnostics**
3. Run the following commands in the CLI to export configuration:

   ```bash
   show ns runningConfig
   show lb vserver
   show service
   show serviceGroup
   show lb monitor
   show ssl certKey
   ```
   {: pre}

4. Save the output for reference during VPC ALB configuration
5. Export SSL certificates and private keys:

   ```bash
   show ssl certKey <cert-name>
   ```
   {: pre}

### Step 2: Prepare SSL certificates
{: #prepare-certificates}

Import your SSL certificates into IBM Cloud Secrets Manager:

1. Export certificates from Citrix VPX (PEM format)
2. Create a Secrets Manager instance if you don't have one
3. Import certificates to Secrets Manager:

   ```bash
   ibmcloud secrets-manager secret-create \
     --secret-type imported_cert \
     --name "my-app-certificate" \
     --certificate @certificate.pem \
     --private-key @private-key.pem \
     --intermediate @intermediate.pem
   ```
   {: pre}

For more information, see [Managing certificates in Secrets Manager](/docs/secrets-manager?topic=secrets-manager-certificates).

### Step 3: Create VPC Application Load Balancer
{: #create-alb}

Create your VPC Application Load Balancer:

#### Using the IBM Cloud Console
{: #create-alb-console}

1. Navigate to **VPC Infrastructure** > **Load balancers**
2. Click **Create**
3. Configure the load balancer:
   * **Name**: Enter a descriptive name
   * **Virtual private cloud**: Select your VPC
   * **Type**: Select **Application Load Balancer**
   * **Subnets**: Select subnets in multiple zones for HA
   * **Resource group**: Select appropriate resource group

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

Create backend pools that correspond to your Citrix VPX service groups:

#### Using the IBM Cloud Console
{: #configure-pools-console}

1. In your load balancer details, navigate to **Back-end pools**
2. Click **Create**
3. Configure the pool:
   * **Name**: Enter a descriptive name (e.g., "web-servers-pool")
   * **Protocol**: Select HTTP or HTTPS
   * **Method**: Choose load balancing algorithm
   * **Session stickiness**: Configure if needed
   * **Health check**: Configure health monitoring

4. Add pool members:
   * Click **Attach** to add VSIs
   * Select target VSIs
   * Specify port and weight

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

Map your Citrix VPX monitors to VPC ALB health checks:

| Health Check Parameter | Description | Recommended Value |
|----------------------|-------------|-------------------|
| **Protocol** | HTTP, HTTPS, or TCP | Match your application |
| **Port** | Backend server port | Same as pool member port |
| **URL Path** | Health check endpoint | /health or /status |
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

### Step 6: Configure frontend listeners
{: #configure-listeners}

Create listeners that correspond to your Citrix VPX virtual servers:

#### HTTP Listener
{: #http-listener}

```bash
ibmcloud is load-balancer-listener-create <load-balancer-id> \
  --port 80 \
  --protocol http \
  --default-pool <pool-id>
```
{: pre}

#### HTTPS Listener with SSL termination
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

If you used Citrix VPX content switching, create listener policies:

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

Configure session affinity to match your Citrix VPX persistence settings:

**HTTP cookie-based persistence**:
```bash
ibmcloud is load-balancer-pool-update <load-balancer-id> <pool-id> \
  --session-persistence-type http_cookie
```
{: pre}

**Source IP persistence** (for TCP/UDP):
```bash
ibmcloud is load-balancer-pool-update <load-balancer-id> <pool-id> \
  --session-persistence-type source_ip
```
{: pre}

### Step 8: Configure security
{: #configure-security}

Secure your VPC ALB with security groups:

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

3. Update backend VSI security groups to allow traffic from ALB subnets

### Step 9: Test the configuration
{: #test-configuration}

Thoroughly test your VPC ALB before cutover:

1. **DNS testing**: Create a test DNS entry pointing to the ALB hostname
2. **Functional testing**: Verify all application features work correctly
3. **Load testing**: Test with production-like traffic volumes
4. **Failover testing**: Verify health checks and automatic failover
5. **SSL/TLS testing**: Verify certificate configuration and cipher suites
6. **Session persistence testing**: Verify sticky sessions work as expected
7. **Performance testing**: Compare response times with Citrix VPX

Use the ALB hostname for testing:
```bash
# Get ALB hostname
ibmcloud is load-balancer <load-balancer-id> --output json | grep hostname
```
{: pre}

### Step 10: Plan and execute cutover
{: #cutover}

Execute the migration cutover:

#### Pre-cutover checklist
{: #pre-cutover-checklist}

- [ ] VPC ALB fully configured and tested
- [ ] All SSL certificates imported and validated
- [ ] Backend pools healthy and responding
- [ ] Monitoring and alerting configured
- [ ] Rollback plan documented
- [ ] Stakeholders notified of maintenance window
- [ ] DNS TTL reduced (e.g., to 300 seconds)

#### Cutover steps
{: #cutover-steps}

1. **Schedule maintenance window**: Choose low-traffic period
2. **Enable parallel operation**: Run both VPX and ALB simultaneously
3. **Update DNS**: Point DNS records to VPC ALB hostname
4. **Monitor traffic**: Watch for errors or performance issues
5. **Verify functionality**: Test critical application paths
6. **Monitor for 24-48 hours**: Keep Citrix VPX running as backup
7. **Decommission VPX**: After successful validation, remove Citrix VPX

#### Rollback procedure
{: #rollback-procedure}

If issues occur:

1. Update DNS to point back to Citrix VPX
2. Wait for DNS propagation (based on TTL)
3. Investigate and resolve issues with VPC ALB
4. Plan another cutover attempt

## Post-migration optimization
{: #post-migration}

### Monitor and tune performance
{: #monitor-performance}

Set up monitoring for your VPC ALB:

1. **Enable IBM Cloud Monitoring**:
   ```bash
   ibmcloud is load-balancer-update <load-balancer-id> \
     --logging-datapath-active true
   ```
   {: pre}

2. **Key metrics to monitor**:
   * Active connections
   * Throughput (bytes/sec)
   * Response time
   * Health check status
   * Backend pool member health
   * SSL/TLS handshake time

3. **Set up alerts** for:
   * All backend members unhealthy
   * High error rates (4xx, 5xx)
   * Connection limits approaching
   * Certificate expiration

### Optimize configuration
{: #optimize-config}

Fine-tune your VPC ALB:

* **Adjust health check intervals**: Balance between responsiveness and backend load
* **Review load balancing algorithm**: Optimize based on actual traffic patterns
* **Update pool member weights**: Distribute load based on capacity
* **Review security group rules**: Ensure least-privilege access
* **Optimize SSL/TLS settings**: Use modern cipher suites and protocols

### Implement high availability
{: #implement-ha}

Enhance resilience:

* **Multi-zone deployment**: Ensure subnets span multiple availability zones
* **Multiple backend pools**: Create redundant pools for critical applications
* **Cross-region failover**: Consider Global Load Balancer for disaster recovery
* **Automated scaling**: Use instance groups with auto-scaling for backend VSIs

### Cost optimization
{: #cost-optimization}

Optimize costs:

* **Right-size backend VSIs**: Monitor utilization and adjust profiles
* **Review data transfer costs**: Optimize for in-region traffic
* **Consolidate load balancers**: Combine multiple applications where appropriate
* **Use private load balancers**: For internal-only applications

## Troubleshooting
{: #troubleshooting}

### Common issues and solutions
{: #common-issues}

**Issue: Backend pool members showing unhealthy**
:   * Verify security group rules allow traffic from ALB subnets
    * Check health check configuration matches backend application
    * Verify backend VSIs are running and accessible
    * Review health check logs in IBM Cloud Monitoring

**Issue: SSL/TLS certificate errors**
:   * Verify certificate is properly imported to Secrets Manager
    * Check certificate expiration date
    * Ensure certificate chain is complete (including intermediates)
    * Verify certificate matches the domain name

**Issue: Session persistence not working**
:   * Verify session persistence type matches application requirements
    * Check if backend application is overriding cookies
    * Review load balancer logs for session routing
    * Test with different browsers/clients

**Issue: Poor performance compared to Citrix VPX**
:   * Review backend VSI performance and sizing
    * Check network latency between ALB and backends
    * Verify health check intervals aren't too aggressive
    * Review load balancing algorithm selection
    * Consider enabling connection pooling on backends

**Issue: Cannot access load balancer**
:   * Verify security group rules allow inbound traffic
    * Check network ACLs on ALB subnets
    * Verify DNS resolution to ALB hostname
    * Review VPC routing tables

For additional troubleshooting, see [Troubleshooting VPC](/docs/vpc?topic=vpc-troubleshooting-vpc).

## Comparison: Feature parity checklist
{: #feature-parity}

Use this checklist to ensure feature parity after migration:

- [ ] All virtual servers (VIPs) migrated to listeners
- [ ] All backend pools configured with correct members
- [ ] Load balancing algorithms match original configuration
- [ ] Health checks configured and functioning
- [ ] SSL/TLS certificates installed and validated
- [ ] Session persistence working as expected
- [ ] Content-based routing rules migrated (if applicable)
- [ ] Monitoring and alerting configured
- [ ] Security policies implemented
- [ ] Performance meets or exceeds baseline
- [ ] Documentation updated

## Next steps
{: #next-steps}

* [Set up monitoring and logging](/docs/vpc?topic=vpc-monitoring-metrics-alb)
* [Configure auto-scaling for backend VSIs](/docs/vpc?topic=vpc-creating-auto-scale-instance-group)
* [Implement Global Load Balancer for multi-region HA](/docs/cis?topic=cis-global-load-balancer-glb-concepts)
* [Review VPC security best practices](/docs/vpc?topic=vpc-security-in-your-vpc)

## Additional resources
{: #additional-resources}

* [VPC Application Load Balancer documentation](/docs/vpc?topic=vpc-load-balancers)
* [VPC Application Load Balancer API reference](/apidocs/vpc#list-load-balancers)
* [IBM Cloud Secrets Manager documentation](/docs/secrets-manager)
* [VPC security groups documentation](/docs/vpc?topic=vpc-using-security-groups)
* [Citrix VPX to cloud-native migration patterns](https://www.ibm.com/think/architectures/patterns)
