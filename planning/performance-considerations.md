---
copyright:
  years: 2026
lastupdated: "2026-10-31"

keywords: vpc migration, classic infrastructure, performance, network architecture, vpc networking, transit gateway, virtual private endpoints

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Performance considerations for migrating from classic infrastructure to VPC
{: #classic-to-vpc-performance}

Migrating from {{site.data.keyword.cloud}} classic infrastructure to {{site.data.keyword.vpc_full}} is fundamentally a network architecture change, not a hardware, or hypervisor migration. Plan for the performance effects of this change.
{: shortdesc}

The compute and storage resources that are available in {{site.data.keyword.vpc_short}} are similar in nature to the resources in classic infrastructure. However, the network model, access controls, and service connectivity patterns are significantly different.

In {{site.data.keyword.cloud_notm}} classic infrastructure, virtual server instances (VSIs) and bare metal servers run on a shared, {{site.data.keyword.IBM}}-managed private network (the `10.0.0.0/8` back-end network). This network provides implicit connectivity between all resources in an account. {{site.data.keyword.IBM_notm}} automatically provisions public and private VLANs. Resources communicate over a flat, globally routed back-end fabric without you needing to configure explicit routing.

{{site.data.keyword.vpc_short}} replaces this model with a logically isolated, software-defined network that you scope to a region and zone, with customer-controlled subnets, route tables, security groups, and network access control lists (ACLs). Resource in different {{site.data.keyword.vpc_short}}s, or even in different subnets, can't communicate without explicit configuration.

The performance implications of this migration mostly affect the network layer. Secondary effects on compute include instance profile availability and virtual network interface (vNIC) capabilities. The migration has minimal direct impact on block storage performance.

## Network architecture changes
{: #c2v-performance-network-architecture-changes}

The network architecture of {{site.data.keyword.vpc_short}} differs from classic infrastructure, and these differences affect performance.

### Classic infrastructure and VPC fundamental differences
{: #c2v-performance-classic-vs-vpc-differences}

Classic infrastructure and {{site.data.keyword.vpc_short}} differ in the following fundamental ways:

| Dimension | Classic infrastructure | {{site.data.keyword.vpc_short}} |
| ----------- | ---------------------- | ----- |
| Network isolation | Shared {{site.data.keyword.IBM_notm}} back-end network (`10.x.x.x`) | Logically isolated per {{site.data.keyword.vpc_short}} |
| IP assignment | {{site.data.keyword.IBM_notm}}-assigned private IP addresses on shared VLANs | Customer-defined CIDR subnets |
| Back-end connectivity | Implicit. All classic resources in an account can communicate. | Explicit. Requires security groups, ACLs, and routing. |
| Public network | Separate public VLAN, per-resource billing | Floating IPs that you attach to a vNIC, and a public gateway for outbound traffic |
| Routing | {{site.data.keyword.IBM_notm}}-managed, implicit through the back-end fabric | Customer-managed route tables |
| Outbound internet (private subnet) | Not required. The public VLAN handles it. | Requires a public gateway attachment |
| Security model | Hardware firewalls and security groups (limited) | Security groups (stateful) and network ACLs (stateless) |
| Network segmentation | VLAN-based | Subnet-based, security group-based, and ACL-based |
{: caption="Classic infrastructure and VPC network architecture comparison" caption-side="bottom"}

### Performance implications of network isolation
{: #c2v-performance-network-isolation-performance}

{{site.data.keyword.cloud_notm}} classic infrastructure uses a three-network architecture: a public network for internet traffic, a private network (`10.x.x.x`) for back-end communication between resources, and a management network for out-of-band access. All classic resources in an account share the private back-end network and can communicate with each other without explicit routing configuration.

{{site.data.keyword.vpc_short}} eliminates this implicit connectivity. You must explicitly permit and route every traffic path. Key performance implications include the following items:

No implicit back-end fabric
:   {{site.data.keyword.vpc_short}} subnets are isolated by default. Traffic between subnets in the same {{site.data.keyword.vpc_short}} traverses the {{site.data.keyword.vpc_short}} implicit router. This traffic typically adds submillisecond overhead, but it is a new routing hop that didn't exist in the shared back-end network of classic infrastructure.

Zone-bound subnets
:   Each {{site.data.keyword.vpc_short}} subnet is bound to a single zone, which is the {{site.data.keyword.cloud_notm}} equivalent of an availability zone. Colocate tightly coupled workloads in the same zone to minimize inter-zone latency (approximately 2 ms between zones in the same region) and avoid cross-zone data transfer charges.

Public gateway for outbound internet
:   Classic VSIs on public VLANs have direct outbound internet access. {{site.data.keyword.vpc_short}} instances in private subnets require a public gateway that you attach to the subnet for outbound internet access. A public gateway is functionally similar to a network address translation (NAT) gateway and adds a small latency overhead (approximately 1-3 ms) for internet-bound traffic.

### Zone placement
{: #c2v-performance-zone-placement}

In classic infrastructure, {{site.data.keyword.IBM_notm}} assigns resources to data centers (for example, `dal10`, `dal12`, and `dal13`), but the back-end fabric spans data centers within a metro. In {{site.data.keyword.vpc_short}}, you explicitly bind each subnet to a zone within a region (for example, `us-south-1`, `us-south-2`, and `us-south-3`).

- Colocate tightly coupled services (for example, application servers and databases) in the same zone to avoid inter-zone latency.
- Review the data center distribution of existing classic workloads and map them to the corresponding {{site.data.keyword.vpc_short}} zones before migration.
- Classic data center names and {{site.data.keyword.vpc_short}} zone names don't map one-to-one in all regions. Verify the mapping in the {{site.data.keyword.cloud_notm}} documentation for your target region.

## Traffic routing and latency
{: #c2v-performance-traffic-routing-latency}

Traffic routing in {{site.data.keyword.vpc_short}} affects latency differently from classic infrastructure.

### Inter-instance communication
{: #c2v-performance-inter-instance-communication}

Classic infrastructure behavior
:   VSIs communicate over the {{site.data.keyword.IBM_notm}} private back-end network by using `10.x.x.x` addresses. All resources in the same {{site.data.keyword.cloud_notm}} account can reach each other on the private network without explicit routing rules. Traffic never leaves the {{site.data.keyword.IBM_notm}} backbone.

{{site.data.keyword.vpc_short}} behavior
:   VSIs communicate through private IP addresses within their subnet. Traffic between subnets traverses the {{site.data.keyword.vpc_short}} implicit router. Traffic between {{site.data.keyword.vpc_short}}s requires explicit peering or {{site.data.keyword.tg_full}}. Security group rules must explicitly permit inter-instance traffic because there is no implicit allow between instances in the same {{site.data.keyword.vpc_short}}.

Performance consideration
:   Applications that are hardcoded to classic private `10.x.x.x` addresses fail in {{site.data.keyword.vpc_short}} because those IP addresses don't carry over. Additionally, you must reexpress any service-to-service calls that rely on implicit classic back-end connectivity with explicit security group rules and verified routing. Update all internal service communication to use {{site.data.keyword.vpc_short}} private IP addresses or {{site.data.keyword.cloud_notm}} private Domain Name System (DNS) hostnames.

### Public gateway latency and throughput
{: #c2v-performance-public-gateway-latency}

For {{site.data.keyword.vpc_short}} instances that require outbound internet access (for example, package updates, external API calls, and {{site.data.keyword.cloud_notm}} service access through public endpoints), you must attach a public gateway to the subnet. The public gateway replaces the implicit public VLAN connectivity of classic infrastructure.

- A public gateway adds approximately 1-3 ms of latency for outbound internet-bound connections that are compared with a classic VSI with a direct public interface.
- A single public gateway serves all instances in its attached subnet. It isn't a per-instance resource.
- A public gateway supports outbound-only connections. For inbound internet-facing traffic, instances require a floating IP, which is a public IP address that is directly attached to the vNIC of the instance. Alternatively, you must route traffic through an {{site.data.keyword.cloud_notm}} Application Load Balancer for {{site.data.keyword.vpc_short}} or {{site.data.keyword.cloud_notm}} Network Load Balancer for {{site.data.keyword.vpc_short}}.

### Floating IPs compared with classic public IPs
{: #c2v-performance-floating-ips-vs-classic}

In classic infrastructure, {{site.data.keyword.IBM_notm}} assigns public IP addresses per interface, and billing depends on public bandwidth usage. In {{site.data.keyword.vpc_short}}, the following conditions apply:

- You allocate floating IPs individually and attach each one to a specific vNIC on a VSI.
- A floating IP enables direct inbound and outbound internet access for a specific instance. Its behavior is similar to a classic public IP address.
- You can attach only one floating IP to each vNIC. For classic VSIs with multiple interfaces (public and private), review whether floating IPs or a load balancer is the appropriate replacement.

Audit all classic VSIs with public interfaces and determine whether each requires a floating IP, a public gateway (outbound only), or a load balancer (inbound, distributed) in {{site.data.keyword.vpc_short}}.

### Cross-VPC and cross-zone latency
{: #c2v-performance-cross-vpc-zone-latency}

In classic infrastructure, the shared back-end fabric means that all resources in an account can communicate with submillisecond latency regardless of data center. {{site.data.keyword.vpc_short}} isolation changes this behavior significantly:

Same {{site.data.keyword.vpc_short}}, same zone
:   Submillisecond latency, which is equivalent to classic back-end performance.

Same {{site.data.keyword.vpc_short}}, cross-zone
:   Approximately 2 ms of latency within the same {{site.data.keyword.cloud_notm}} region. Cross-zone data transfer incurs per-GB charges.

Cross-{{site.data.keyword.vpc_short}} (same region)
:   Requires {{site.data.keyword.tg_full_notm}} or {{site.data.keyword.vpc_short}} peering. Adds a routing hop with approximately 1-2 ms of overhead.

Cross-region
:   Requires {{site.data.keyword.tg_short}} with cross-region routing. Latency depends on geographic distance.

## Security groups and access control lists
{: #c2v-performance-security-groups-acls}

Security groups and ACLs in {{site.data.keyword.vpc_short}} differ from the security controls in classic infrastructure, and these differences can affect performance.

### Security group behavior differences
{: #c2v-performance-security-group-differences}

Security group behavior differs between classic infrastructure and {{site.data.keyword.vpc_short}} in the following ways:

| Behavior | Classic infrastructure | {{site.data.keyword.vpc_short}} |
| ---------- | ---------------------- | ----- |
| Type | Stateful | Stateful |
| Scope | Applied per instance | Applied per vNIC |
| Default inbound | Deny all | Deny all |
| Default outbound | Allow all | Allow all |
| Rule references | IP address or CIDR only | IP address, CIDR, or security group ID |
| Maximum rules | Varies by firewall type | 50 inbound and 50 outbound (default, adjustable) |
| Hardware firewalls | Available as an add-on | Not available. Replaced by ACLs and security groups. |
{: caption="Security group behavior comparison" caption-side="bottom"}

### Security migration considerations
{: #c2v-performance-security-migration-considerations}

Hardware firewalls
:   Classic infrastructure supports dedicated hardware firewalls (FortiGate&reg; Security Appliance and Juniper&reg; vSRX) and shared firewalls as add-on services. These firewalls don't exist in {{site.data.keyword.vpc_short}}. You must translate all hardware firewall rules into {{site.data.keyword.vpc_short}} security group rules and network ACL rules before migration.

Rule translation complexity
:   Classic hardware firewall rules are often more expressive (for example, stateful inspection and application-layer filtering). {{site.data.keyword.vpc_short}} security groups are stateful but operate at the network layer only. Evaluate whether any classic hardware firewall capabilities require a third-party network virtual appliance (NVA) in {{site.data.keyword.vpc_short}} for equivalent functions.

Performance impact of rule complexity
:   Security groups in {{site.data.keyword.vpc_short}} are evaluated per vNIC. Instances with many attached security groups and large rule sets might exhibit marginally higher connection setup latency. Keep rule sets lean by using one security group per functional role (web tier, app tier, and data tier) rather than a single large group.

### Network ACLs
{: #c2v-performance-network-acls}

{{site.data.keyword.vpc_short}} introduces network ACLs as a stateless, subnet-level access control layer that has no direct equivalent in classic infrastructure. Hardware firewalls serve a similar but separate purpose. Unlike security groups, network ACLs have the following characteristics:

- ACLs are stateless. You must explicitly configure both inbound and outbound rules for bidirectional traffic. ACLs don't automatically permit return traffic.
- The system evaluates rules in ascending order by priority number and applies the first matching rule.
- ACLs apply to all instances within a subnet, not per instance.
- Each subnet can have only one ACL. You can associate one ACL with multiple subnets.

ACLs with many rules add stateless evaluation overhead for every packet at high connection rates. Use ACLs for coarse-grained subnet boundary control (for example, block all traffic from outside CIDR ranges) and rely on security groups for fine-grained instance-level filtering. Help ensure that the ephemeral port range (1024-65535) is open in outbound ACL rules to allow return traffic for inbound-initiated connections.

## DNS, hostname, and IP address behavior
{: #c2v-performance-dns-hostname-ip-behavior}

DNS, hostname, and IP address behavior changes when you migrate to {{site.data.keyword.vpc_short}}.

### Hostname and IP address format changes
{: #c2v-performance-hostname-ip-changes}

Hostname and IP address behavior differs between classic infrastructure and {{site.data.keyword.vpc_short}} in the following ways:

| Context | Classic infrastructure | {{site.data.keyword.vpc_short}} |
| --------- | ---------------------- | ----- |
| Private hostname | `<hostname>.private` (customer-defined) | `<hostname>.<vpc>.internal` ({{site.data.keyword.IBM_notm}}-assigned) |
| Private IP address | {{site.data.keyword.IBM_notm}}-assigned from the shared `10.x.x.x` pool | Customer-defined CIDR subnet |
| Public hostname | Customer-defined or {{site.data.keyword.IBM_notm}}-assigned fully qualified domain name (FQDN) | Assigned when a floating IP is attached |
| IP address persistence | Private IP address is retained while provisioned | A private IP address is retained while the VSI exists (tied to the vNIC) |
| DNS resolver | {{site.data.keyword.IBM_notm}} private DNS at `10.0.80.11` and `10.0.80.12` | {{site.data.keyword.cloud_notm}} DNS Services ({{site.data.keyword.vpc_short}}-native, zone-scoped) |
{: caption="Hostname and IP address format comparison" caption-side="bottom"}

### DNS migration considerations
{: #c2v-performance-dns-migration-considerations}

Hardcoded classic private IP addresses
:   Classic VSI private IP addresses come from the shared `10.x.x.x` address space of {{site.data.keyword.IBM_notm}} and aren't portable to {{site.data.keyword.vpc_short}}. After migration, you must update all application configuration files, connection strings, and service discovery mechanisms that reference classic private IP addresses to use {{site.data.keyword.vpc_short}} private IP addresses.

DNS resolution differences
:   Classic infrastructure uses {{site.data.keyword.IBM_notm}}-managed private DNS resolvers that are accessible at well-known `10.x.x.x` addresses. {{site.data.keyword.vpc_short}} uses {{site.data.keyword.cloud_notm}} DNS Services, which you configure per {{site.data.keyword.vpc_short}} and which resolves names within the DNS zone of the {{site.data.keyword.vpc_short}}. Applications that rely on classic private DNS resolution (`*.private`) don't resolve correctly in {{site.data.keyword.vpc_short}} without DNS configuration updates.

Custom DNS zones
:   If your classic environment uses custom private DNS zones (through {{site.data.keyword.cloud_notm}} DNS Services or self-hosted BIND), you must re-create these zones and associate them with the target {{site.data.keyword.vpc_short}} before migration. Verify that all internal hostnames resolve correctly in {{site.data.keyword.vpc_short}} before you cut over traffic.

Public IP address reallocation
:   You can't migrate classic public IP addresses (primary and portable subnets) directly to {{site.data.keyword.vpc_short}}. You must allocate new floating IPs from the {{site.data.keyword.vpc_short}} pool. Update all external DNS records, third-party allowlists, and firewall rules that reference classic public IP addresses before or immediately after cutover.

## Load balancer performance considerations
{: #c2v-performance-load-balancer-performance}

{{site.data.keyword.vpc_short}} load balancers have distinct performance characteristics.

### Application Load Balancer differences
{: #c2v-performance-alb-differences}

{{site.data.keyword.vpc_short}} Application Load Balancers (ALBs) have different performance characteristics that are compared with classic infrastructure load balancers:

Connection limits
:   {{site.data.keyword.vpc_short}} ALBs support up to 15,000 concurrent connections per member. Classic load balancers have different connection limits, depending on the service tier.

SSL/TLS termination performance
:   By default, {{site.data.keyword.vpc_short}} ALBs use modern cipher suites and TLS 1.2 and 1.3, which can provide better performance than older classic load balancer configurations.

Health check impact
:   By default, {{site.data.keyword.vpc_short}} ALB health checks run every 5 seconds. More frequent health checks provide faster failure detection but increase back-end instance load. Adjust health check intervals based on application requirements.

Layer 7 routing
:   {{site.data.keyword.vpc_short}} ALBs support advanced Layer 7 routing (host-based and path-based) with minimal performance overhead that is compared with classic load balancers.

### Network Load Balancer for high-throughput workloads
{: #c2v-performance-nlb-high-throughput}

For workloads that require maximum throughput and minimum latency, {{site.data.keyword.vpc_short}} Network Load Balancers (NLBs) operate at Layer 4:

- NLBs support millions of requests per second with submillisecond latency.
- Direct server return (DSR) mode preserves client IP addresses without NAT overhead.
- NLBs are zone-specific. Deploy one NLB per zone for high availability.
- NLBs don't provide SSL/TLS termination. Traffic passes through to back-end instances.

### Load balancer placement and zone considerations
{: #c2v-performance-lb-placement}

Consider the following guidelines for load balancer placement:

- Deploy load balancers in the same zone as the back-end instances to minimize latency.
- For multizone deployments, use one load balancer per zone or a global load balancer ({{site.data.keyword.cis_full}}).
- Cross-zone load balancing adds approximately 2 ms of latency per request.
- Monitor the back-end instance distribution to avoid zone imbalance.

## Connectivity: VPC peering, Transit Gateway, and Direct Link
{: #c2v-performance-connectivity-options}

You can connect {{site.data.keyword.vpc_short}} resources to each other, to classic infrastructure, and to on-premises networks by using several options.

### Replacing implicit back-end connectivity from classic infrastructure
{: #c2v-performance-replacing-classic-connectivity}

The shared private backbone of classic infrastructure means that all resources in an {{site.data.keyword.cloud_notm}} account can communicate with each other without explicit configuration. This connectivity extends across data centers, VLANs, and even some classic services. {{site.data.keyword.vpc_short}} isolation eliminates this implicit connectivity. You must explicitly connect workloads that span multiple classic data centers and rely on the back-end fabric for inter-service communication.

### {{site.data.keyword.tg_full_notm}}
{: #c2v-performance-transit-gateway}

{{site.data.keyword.tg_full_notm}} is the primary mechanism for connecting multiple {{site.data.keyword.vpc_short}}s and classic infrastructure environments in {{site.data.keyword.cloud_notm}}.

- It connects multiple {{site.data.keyword.vpc_short}}s (in the same region or different regions) and classic infrastructure networks through a single gateway.
- It supports transitive routing. Unlike direct {{site.data.keyword.vpc_short}} peering, {{site.data.keyword.tg_short}} allows A-to-B-to-C communication through the hub.
- It supports classic infrastructure connections. During a phased migration, {{site.data.keyword.tg_short}} can bridge classic and {{site.data.keyword.vpc_short}} environments, which allows gradual workload migration without a hard cutover.
- It adds approximately 1-2 ms of routing overhead that is compared with intra-{{site.data.keyword.vpc_short}} communication.

Classic-to-{{site.data.keyword.vpc_short}} coexistence through {{site.data.keyword.tg_short}} is the recommended migration pattern for complex environments. This pattern allows classic and {{site.data.keyword.vpc_short}} workloads to communicate during the transition period without you needing to migrate all services simultaneously.

### VPC address prefix and routing considerations
{: #c2v-performance-vpc-address-prefix-routing}

{{site.data.keyword.vpc_short}} uses address prefixes to define the IP address ranges that are routable within a {{site.data.keyword.vpc_short}}. Help ensure that you meet the following conditions:

- {{site.data.keyword.vpc_short}} address prefixes don't overlap with classic infrastructure `10.x.x.x` ranges if you use {{site.data.keyword.tg_short}} to bridge both environments. Overlapping CIDRs causes routing conflicts.
- You configure ingress routing tables if you need to direct traffic from {{site.data.keyword.tg_short}} or {{site.data.keyword.dl_full}} to a specific next hop (for example, a network virtual appliance for inspection).

### {{site.data.keyword.dl_full_notm}}
{: #c2v-performance-direct-link}

If your classic environment uses {{site.data.keyword.dl_full_notm}} for on-premises hybrid connectivity, note the following information:

- Classic {{site.data.keyword.dl_short}} connections aren't automatically available in {{site.data.keyword.vpc_short}}.
- You must provision {{site.data.keyword.dl_short}} 2.0 (the {{site.data.keyword.vpc_short}}-native offering) separately and connect it to the {{site.data.keyword.vpc_short}} through {{site.data.keyword.tg_short}} or directly.
- {{site.data.keyword.dl_short}} 2.0 supports speeds from 50 Mbps to 10 Gbps and provides dedicated, private connectivity between on-premises infrastructure and {{site.data.keyword.vpc_short}} with predictable latency.

### Connectivity performance comparison
{: #c2v-performance-connectivity-performance-comparison}

Performance characteristics differ for each connectivity path:

| Path | Latency | Throughput | Cost |
| ------ | --------- | ----------- | ------ |
| Same {{site.data.keyword.vpc_short}}, same zone | Less than 1 ms | Instance bandwidth limit | None |
| Same {{site.data.keyword.vpc_short}}, cross-zone | Approximately 2 ms | Instance bandwidth limit | Per-GB transfer fee |
| Cross-{{site.data.keyword.vpc_short}} through {{site.data.keyword.tg_short}} (same region) | Approximately 1-2 ms of overhead | {{site.data.keyword.tg_short}} aggregate limit | Per-GB fee and connection fee |
| Cross-region through {{site.data.keyword.tg_short}} | Geographic latency | {{site.data.keyword.tg_short}} limit | Per-GB fee (higher rate) |
| On-premises through {{site.data.keyword.dl_short}} 2.0 | Dedicated circuit latency | Up to 10 Gbps | Per-GB egress fee |
{: caption="Performance characteristics of connectivity paths" caption-side="bottom"}

## Compute considerations
{: #c2v-performance-compute-considerations}

Compute resources in {{site.data.keyword.vpc_short}} differ from compute resources in classic infrastructure.

### Instance profile availability
{: #c2v-performance-instance-profile-availability}

{{site.data.keyword.cloud_notm}} classic infrastructure offers VSI flavors (fixed configurations) and bare metal server profiles in a shared, multitenant environment. {{site.data.keyword.vpc_short}} organizes instance profiles into families.

| {{site.data.keyword.vpc_short}} profile family | Description | Equivalent classic use case |
| -------------------- | ------------- | --------------------------- |
| Balanced (`bx2`) | Equal vCPU-to-RAM ratio | General-purpose classic VSIs |
| Compute (`cx2`) | Higher vCPU-to-RAM ratio | CPU-intensive classic VSIs |
| Memory (`mx2`) | Higher RAM-to-vCPU ratio | Memory-intensive classic VSIs |
| Ultra high memory (`ux2`) | Very high RAM ratios | In-memory database workloads |
| GPU (`gx2`) | NVIDIA&reg; GPU-attached | GPU-enabled classic VSIs |
| Very high memory (`vx2`) | SAP&reg;-certified profiles | SAP HANA on classic infrastructure |
{: caption="VPC instance profile families" caption-side="bottom"}

Not all classic VSI configurations map directly to a {{site.data.keyword.vpc_short}} profile. Classic infrastructure offers highly flexible custom vCPU, RAM, and disk combinations, but {{site.data.keyword.vpc_short}} profiles are fixed configurations. Audit all classic VSI configurations and identify the closest {{site.data.keyword.vpc_short}} profile match. This audit might also be an opportunity to right-size.

### Enhanced networking in VPC
{: #c2v-performance-enhanced-networking}

{{site.data.keyword.vpc_short}} VSIs use single root I/O virtualization (SR-IOV) for network I/O. SR-IOV provides hardware-level network virtualization that bypasses the hypervisor for network traffic. Classic VSIs use software-based virtual networking.

The practical performance differences are as follows:

Lesser CPU overhead
:   The host CPU is less involved in packet processing, which frees vCPUs for application workloads.

Higher packet-per-second (PPS) throughput
:   Network-intensive workloads benefit from increased PPS capacity.

Lesser and more consistent network latency
:   {{site.data.keyword.vpc_short}} instances demonstrate improved latency that is compared with classic VSIs on equivalent hardware.

For network-intensive workloads (for example, distributed databases, messaging systems, and real-time data pipelines), this change is one of the most tangible performance improvements available from a classic-to-{{site.data.keyword.vpc_short}} migration.

After migration, run the following commands to validate the network improvement:

```bash
# Measure inter-instance throughput (run on source and target VSI)
iperf3 -s  # on receiver
iperf3 -c <target-private-ip> -t 30 -P 8  # on sender; 8 parallel streams

# Measure latency
ping <target-private-ip> -c 100

# Confirm SR-IOV vNIC driver (Linux)
ethtool -i eth0 | grep driver
# Expected on VPC: virtio_net or ena depending on image
```
{: codeblock}

## Migrating from classic bare metal to VPC VSI
{: #c2v-performance-bare-metal-to-vsi-migration}

Migrating workloads from classic bare metal servers to {{site.data.keyword.vpc_short}} virtual server instances is a significant architectural change that has important performance implications. Bare metal servers provide dedicated physical resources. {{site.data.keyword.vpc_short}} VSIs offer flexibility, faster provisioning, and modern virtualization benefits.

### Performance comparison of bare metal servers and VPC VSIs
{: #c2v-performance-bare-metal-vsi-comparison}

Key performance characteristics differ between classic bare metal servers and {{site.data.keyword.vpc_short}} VSIs:

| Characteristic | Classic bare metal | {{site.data.keyword.vpc_short}} VSI |
| ---------------- | ------------------- | --------- |
| CPU performance | Dedicated physical cores, no hypervisor overhead | Virtual CPUs with approximately 5-10% virtualization overhead |
| Memory performance | Direct physical memory access | Virtual memory with minimal overhead (less than 2%) |
| Network performance | 10-20 Gbps typical, software-based | Up to 80 Gbps with SR-IOV, hardware-accelerated |
| Storage I/O | Direct-attached local disks or storage area network (SAN) | Network-attached block storage with consistent performance |
| Provisioning time | 2-4 hours | 2-5 minutes |
| Scaling flexibility | Manual hardware provisioning | Automated scaling with instance groups |
| Cost model | Monthly fixed cost | Hourly billing with autoscaling |
{: caption="Classic bare metal and VPC VSI performance comparison" caption-side="bottom"}

### CPU performance considerations
{: #c2v-performance-cpu-performance-bare-metal-vsi}

Workload characterization
:   CPU-bound workloads with sustained 100% utilization might experience a 5-10% performance reduction on {{site.data.keyword.vpc_short}} VSIs because of virtualization overhead. Evaluate whether the workload requires dedicated physical cores or whether the flexibility of {{site.data.keyword.vpc_short}} outweighs the marginal performance difference.

vCPU-to-physical-core mapping
:   {{site.data.keyword.vpc_short}} VSIs use hyperthreading. A 16-vCPU instance typically maps to 8 physical cores with 2 threads each. Bare metal servers provide direct access to physical cores without hyperthreading overhead.

CPU pinning and nonuniform memory access (NUMA)
:   Classic bare metal servers allow explicit CPU pinning and control of NUMA nodes for latency-sensitive workloads. {{site.data.keyword.vpc_short}} VSIs don't expose the NUMA topology, but the hypervisor optimizes placement automatically.

Performance validation
:   Before migration, benchmark CPU-intensive workloads in {{site.data.keyword.vpc_short}}. Use tools such as `sysbench`, `stress-ng`, or application-specific benchmarks to compare performance under realistic load.

### Memory performance and capacity
{: #c2v-performance-memory-performance-bare-metal-vsi}

Memory overhead
:   {{site.data.keyword.vpc_short}} VSIs have minimal memory virtualization overhead (less than 2%). Most workloads show no measurable difference in memory performance that is compared with bare metal.

Memory capacity limits
:   Classic bare metal servers support up to 1.5 TB of RAM on high-memory configurations. {{site.data.keyword.vpc_short}} Ultra high memory (`ux2`) profiles support up to 896 GB of RAM. For workloads that require more than 896 GB of RAM, consider {{site.data.keyword.vpc_short}} bare metal servers or data partitioning strategies.

Memory-intensive workloads
:   In-memory databases (for example, Redis&reg;, Memcached, and SAP HANA) perform well on {{site.data.keyword.vpc_short}} VSIs if the required memory capacity fits within the {{site.data.keyword.vpc_short}} profile limits. Use `mx2` or `ux2` profiles for memory-intensive workloads.

Huge pages
:   {{site.data.keyword.vpc_short}} VSIs support transparent huge pages (THP) for improved memory performance. To enable THP for database and analytics workloads, run `echo always > /sys/kernel/mm/transparent_hugepage/enabled`.

### Network performance: SR-IOV advantage
{: #c2v-performance-network-performance-bare-metal-vsi}

Network performance is one of the most significant performance improvements when you migrate from classic bare metal servers to {{site.data.keyword.vpc_short}} VSIs:

Throughput comparison
:   Classic bare metal servers typically provide 10-20 Gbps of network bandwidth with software-based networking. {{site.data.keyword.vpc_short}} VSIs with SR-IOV provide up to 80 Gbps on larger profiles (for example, `bx2-48x192`), with hardware-accelerated packet processing.

Latency improvement
:   SR-IOV bypasses the hypervisor for network I/O, which results in lower and more consistent latency than the software-based networking of classic bare metal servers. The typical improvement is 20-30% lower latency for inter-instance communication.

PPS performance
:   {{site.data.keyword.vpc_short}} VSIs with SR-IOV handle significantly higher PPS rates than classic bare metal servers. This capability benefits microservices architectures, real-time data processing, and high-frequency trading applications.

Network-intensive workload recommendation
:   For network-bound workloads (for example, web servers, API gateways, load balancers, and distributed databases), {{site.data.keyword.vpc_short}} VSIs often outperform classic bare metal servers despite the virtualization layer.

### Storage performance considerations
{: #c2v-performance-storage-performance-bare-metal-vsi}

Local disk compared with network storage
:   Classic bare metal servers use local Serial Advanced Technology Attachment (SATA) or Serial Attached Small Computer System Interface (SAS) disks or direct-attached Non-Volatile Memory Express (NVMe). {{site.data.keyword.vpc_short}} VSIs use network-attached {{site.data.keyword.block_storage_is_full}}. Network storage adds latency (approximately 1-2 ms), but {{site.data.keyword.vpc_short}} block storage provides consistent input/output operations per second (IOPS) and throughput regardless of instance size.

IOPS and throughput
:   {{site.data.keyword.vpc_short}} block storage offers tiered performance: 3 IOPS/GB (general purpose), 5 IOPS/GB (`5iops-tier`), 10 IOPS/GB (`10iops-tier`), and custom IOPS of up to 48,000 per volume. The local disk performance of classic bare metal servers varies by disk type and redundant array of independent disks (RAID) configuration.

Storage I/O patterns
:   Sequential I/O workloads (for example, large file processing and backups) perform well on {{site.data.keyword.vpc_short}} block storage. Random I/O workloads (for example, databases and transactional systems) benefit from higher IOPS tiers or custom IOPS volumes.

NVMe local storage alternative
:   For workloads that require local NVMe performance, consider {{site.data.keyword.vpc_short}} bare metal servers, which provide local NVMe storage with {{site.data.keyword.vpc_short}} networking benefits.

### Workload suitability assessment
{: #c2v-performance-workload-suitability-assessment}

The following workload types are well suited for migration from classic bare metal servers to {{site.data.keyword.vpc_short}} VSIs:

Suitable workloads
:   Web applications, API services, microservices, containerized workloads, development and test environments, CI/CD pipelines, general-purpose databases, caching layers, message queues, and batch processing jobs.

Workloads that require evaluation
:   High-frequency trading systems, real-time analytics with sustained 100% CPU, in-memory databases with more than 896 GB of RAM, applications with strict NUMA requirements, and workloads with specialized hardware dependencies.

Alternative: {{site.data.keyword.vpc_short}} bare metal servers
:   For workloads that require dedicated physical resources, consider {{site.data.keyword.vpc_short}} bare metal servers, which combine bare metal performance with {{site.data.keyword.vpc_short}} networking, security groups, and modern cloud features.

### Migration strategy and performance validation
{: #c2v-performance-migration-strategy-validation}

Phased migration approach
:   Migrate noncritical workloads first to validate performance in the {{site.data.keyword.vpc_short}} environment. Use classic-to-{{site.data.keyword.vpc_short}} coexistence through {{site.data.keyword.tg_short}} to enable gradual migration without a hard cutover.

Performance baseline establishment
:   Before migration, establish performance baselines on classic bare metal servers: CPU utilization patterns, memory usage, network throughput, storage IOPS, and application response times.

{{site.data.keyword.vpc_short}} performance testing
:   Deploy pilot workloads on {{site.data.keyword.vpc_short}} VSIs and run identical benchmarks. Compare the results against the classic baselines to identify performance gaps or improvements.

Profile selection optimization
:   Start with {{site.data.keyword.vpc_short}} profiles that match or exceed the specifications of the classic bare metal servers. After performance testing, right-size to optimal profiles based on actual resource utilization.

Monitoring and tuning
:   After migration, use {{site.data.keyword.mon_full}} to monitor {{site.data.keyword.vpc_short}} VSI performance. Tune TCP parameters, enable huge pages, and optimize application configurations for the virtualized environment.

### Cost-performance tradeoffs
{: #c2v-performance-cost-performance-tradeoffs}

Total cost of ownership
:   While individual {{site.data.keyword.vpc_short}} VSI hourly costs might appear higher than amortized bare metal costs, consider other factors. These factors include faster provisioning (minutes compared with hours), autoscaling capabilities, pay-per-use billing, elimination of overprovisioning, and reduced operational overhead.

Performance per dollar
:   {{site.data.keyword.vpc_short}} VSIs with SR-IOV networking often provide better performance per dollar for network-intensive workloads compared with classic bare metal servers with software-based networking.

Reserved capacity
:   For predictable workloads, {{site.data.keyword.vpc_short}} reserved capacity (1-year or 3-year terms) provides up to a 30% discount, which makes {{site.data.keyword.vpc_short}} VSIs cost-competitive with classic bare metal servers while you maintain flexibility.

Scaling economics
:   The ability of {{site.data.keyword.vpc_short}} to scale horizontally (add more VSIs) compared with scaling vertically (larger bare metal servers) often provides better cost-performance optimization for modern cloud-native applications.

### Dedicated hosts
{: #c2v-performance-dedicated-hosts}

{{site.data.keyword.vpc_short}} supports dedicated hosts. A dedicated host is a single-tenant physical host that only your account uses. Classic infrastructure also offers single-tenant dedicated instances, but the {{site.data.keyword.vpc_short}} dedicated host model provides more explicit control over the following items:

- Physical host selection and placement
- License compliance (bring-your-own-license workloads that require physical core counting)
- Workload isolation from other tenants for regulated industries

If your classic environment used dedicated instances for compliance or licensing reasons, map these instances to {{site.data.keyword.vpc_short}} dedicated hosts to maintain equivalent isolation guarantees.

### Instance bandwidth caps
{: #c2v-performance-instance-bandwidth-caps}

{{site.data.keyword.vpc_short}} instance profiles have explicit network bandwidth allocations per profile (for example, a `bx2-4x16` profile is allocated up to 8 Gbps). Classic VSIs have shared network access without explicit per-instance caps.

Review the bandwidth allocation for each target {{site.data.keyword.vpc_short}} profile and confirm that it meets the peak network throughput requirements of the workload. For bandwidth-intensive workloads, select a larger profile or a profile family with a higher network allocation.

## Autoscaling and provisioning performance
{: #c2v-performance-autoscaling-provisioning}

Provisioning time affects autoscaling in {{site.data.keyword.vpc_short}}.

### Instance provisioning time
{: #c2v-performance-instance-provisioning-time}

{{site.data.keyword.vpc_short}} instance provisioning time affects autoscaling responsiveness:

Boot time comparison
:   {{site.data.keyword.vpc_short}} instances typically boot in 2-5 minutes, depending on image size and profile. Classic VSIs have similar boot times, but SR-IOV network initialization in {{site.data.keyword.vpc_short}} adds 10-30 seconds.

Custom image optimization
:   Smaller custom images (under 10 GB) provision faster than larger images. Optimize images by removing unnecessary packages and by using `cloud-init` for post-boot configuration.

Stock images compared with custom images
:   {{site.data.keyword.IBM_notm}} caches its stock images in each zone, so they provision faster than custom images, which the system retrieves from {{site.data.keyword.cos_full}}.

### Autoscaling response time
{: #c2v-performance-autoscaling-response}

Instance groups in {{site.data.keyword.vpc_short}} support autoscaling based on CPU, memory, or network utilization:

- Scaling decisions occur every 60 seconds (configurable).
- The time from scaling trigger to a ready instance is 3-7 minutes. This time includes provisioning, boot, and health check.
- To scale up faster, prewarm your environment by maintaining a buffer of running instances.
- To prevent thrashing during variable load, configure cooldown periods for scale-down.

### Image migration and availability
{: #c2v-performance-image-migration}

Consider the following information about image migration and availability:

- You must export classic custom images and import them to {{site.data.keyword.vpc_short}} before migration.
- Image import from {{site.data.keyword.cos_short}} takes 10-30 minutes, depending on image size.
- Store custom images in the same region as the target {{site.data.keyword.vpc_short}} to minimize import time.
- Use image lifecycle policies to manage image versions and reduce storage costs.

## Bare metal servers for VPC
{: #c2v-performance-bare-metal-vpc}

{{site.data.keyword.vpc_short}} bare metal servers have distinct performance characteristics.

### Performance profile differences
{: #c2v-performance-bare-metal-profiles}

{{site.data.keyword.vpc_short}} bare metal servers have different characteristics from classic bare metal servers:

Network performance
:   {{site.data.keyword.vpc_short}} bare metal servers use SR-IOV networking with up to 200 Gbps of bandwidth on supported profiles. Classic bare metal servers typically provide 10-20 Gbps.

Storage options
:   {{site.data.keyword.vpc_short}} bare metal servers support local NVMe storage and network-attached {{site.data.keyword.block_storage_is_short}}. Local NVMe provides lower latency than classic local disks.

Provisioning time
:   {{site.data.keyword.vpc_short}} bare metal server provisioning takes 30-90 minutes, which is similar to the provisioning time of classic bare metal servers.

### Network performance optimization
{: #c2v-performance-bare-metal-network-optimization}

Consider the following guidelines to optimize network performance:

- Use VLAN interfaces for network segmentation without a performance penalty.
- Configure jumbo frames (maximum transmission unit (MTU) of 9000) for storage and replication traffic.
- Use multiple network interfaces for traffic separation.
- Monitor network utilization to identify bottlenecks.

### VLAN spanning equivalent
{: #c2v-performance-vlan-spanning-equivalent}

In classic infrastructure, VLAN spanning allows bare metal servers to communicate across VLANs. In {{site.data.keyword.vpc_short}}, note the following information:

- Use multiple network interfaces that are attached to different subnets.
- Configure routing between subnets by using {{site.data.keyword.vpc_short}} routing tables.
- Security groups and ACLs control traffic between subnets.
- There is no implicit connectivity. You must explicitly configure all paths.

## Storage considerations
{: #c2v-performance-storage-considerations}

The performance characteristics of block storage ({{site.data.keyword.block_storage_is_short}}, which is equivalent to classic Block Storage) aren't materially changed by the network architecture migration. However, two storage-related network topics require attention.

### {{site.data.keyword.cos_full_notm}} access path
{: #c2v-performance-cos-access-path}

In classic infrastructure, you access {{site.data.keyword.cos_short}} through private endpoints over the classic backbone at no transfer cost and with less latency. In {{site.data.keyword.vpc_short}}, if you don't configure a virtual private endpoint (VPE) for {{site.data.keyword.cos_short}}, the following conditions apply:

- Traffic routes through the public internet by way of a public gateway or floating IP, which adds latency and incurs egress charges.
- For workloads that make heavy use of {{site.data.keyword.cos_short}} (for example, data pipelines, log archiving, backup, and media storage), this path change can significantly affect throughput and cost.

Configure a VPE for {{site.data.keyword.cos_full_notm}} immediately after {{site.data.keyword.vpc_short}} provisioning to restore private, less-latency, no-egress-cost {{site.data.keyword.cos_short}} access.

### {{site.data.keyword.filestorage_vpc_short}}
{: #c2v-performance-file-storage-vpc}

Classic infrastructure uses Endurance and Performance Network File System (NFS) file storage that {{site.data.keyword.IBM_notm}} provisions from its storage fabric. {{site.data.keyword.vpc_short}} uses {{site.data.keyword.filestorage_vpc_full}} (NFS-based, zonal). Key differences include the following items:

- {{site.data.keyword.filestorage_vpc_short}} is zone-scoped. You provision a file share within a specific zone, and it is accessible only from that zone (or across zones through share replication).
- You create mount targets within a specific subnet, and security groups that you apply to the mount target control access.
- Help ensure that the consuming VSIs and the file share mount target are in the same zone to avoid cross-zone latency and transfer charges.
- Classic NFS mounts that use `10.x.x.x` storage endpoints don't resolve in {{site.data.keyword.vpc_short}}. You must provision new mount targets and update mount commands.

## {{site.data.keyword.cloud_notm}} service endpoints and private connectivity
{: #c2v-performance-service-endpoints-private-connectivity}

The access path to {{site.data.keyword.cloud_notm}} services changes when you migrate to {{site.data.keyword.vpc_short}}.

### Access to {{site.data.keyword.cloud_notm}} services in classic infrastructure and VPC
{: #c2v-performance-classic-vs-vpc-service-access}

In classic infrastructure, {{site.data.keyword.cloud_notm}} services ({{site.data.keyword.cos_short}}, {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.keymanagementservicelong}}, and others) are accessible through private service endpoints over the classic private backbone (`10.x.x.x`). These endpoints incur no additional network cost, and traffic doesn't traverse the public internet.

In {{site.data.keyword.vpc_short}}, the path to {{site.data.keyword.cloud_notm}} services changes depending on configuration:

Without virtual private endpoints (VPEs)
:   Traffic to {{site.data.keyword.cloud_notm}} services must traverse the public internet through a floating IP or public gateway, which adds latency and exposes traffic unnecessarily.

With VPEs
:   Traffic stays on the {{site.data.keyword.cloud_notm}} private network, and the service endpoint appears as a private IP address within your {{site.data.keyword.vpc_short}} subnet. Performance is equivalent to or better than classic private endpoint performance.

### Virtual private endpoints
{: #c2v-performance-virtual-private-endpoints}

{{site.data.keyword.cloud_notm}} Virtual Private Endpoints for {{site.data.keyword.vpc_short}} create a private IP address within your {{site.data.keyword.vpc_short}} that maps to an {{site.data.keyword.cloud_notm}} service, so all traffic stays on the private backbone.

VPEs support the following common {{site.data.keyword.cloud_notm}} services:

| Service category | Examples | VPE support |
| ----------------- | ---------- | ------------- |
| Object Storage | {{site.data.keyword.cos_full_notm}} | Yes |
| Databases | {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-mysql}}, {{site.data.keyword.databases-for-redis}}, {{site.data.keyword.databases-for-mongodb}}, {{site.data.keyword.databases-for-elasticsearch}} | Yes |
| Key management | {{site.data.keyword.keymanagementserviceshort}}, {{site.data.keyword.hscrypto}} | Yes |
| Container Registry | {{site.data.keyword.registrylong}} | Yes |
| DevOps | {{site.data.keyword.contdelivery_full}}, Toolchain | Yes |
| Observability | {{site.data.keyword.la_full}}, {{site.data.keyword.mon_short}} | Yes |
{: caption="VPE support for {{site.data.keyword.cloud_notm}} services" caption-side="bottom"}

### Recommendations for VPEs
{: #c2v-performance-vpe-recommendations}

- Configure VPEs for all {{site.data.keyword.cloud_notm}} services that your workload consumes before migration. Without VPEs, service traffic routes through the public internet, which adds latency, cost, and security exposure.
- VPEs create a reserved IP address within your chosen subnet. Plan the subnet IP address space to accommodate one reserved IP address per VPE per zone.
- Unlike classic private endpoints, which are global within an account, VPEs are {{site.data.keyword.vpc_short}}-scoped. A VPE that you create in one {{site.data.keyword.vpc_short}} isn't accessible from another {{site.data.keyword.vpc_short}} without {{site.data.keyword.tg_short}} or {{site.data.keyword.vpc_short}} peering.
- Update application endpoint URLs from classic private endpoint hostnames (for example, `s3.private.us-south.cloud-object-storage.appdomain.cloud`) to VPE-resolved private DNS names.

## Kubernetes and container workload performance
{: #c2v-performance-kubernetes-container-performance}

Kubernetes&reg; clusters on {{site.data.keyword.vpc_short}} have distinct performance characteristics.

### {{site.data.keyword.containerlong_notm}} on VPC compared with classic infrastructure
{: #c2v-performance-iks-vpc-vs-classic}

{{site.data.keyword.containerlong}} on {{site.data.keyword.vpc_short}} has different network performance characteristics:

Pod networking
:   {{site.data.keyword.vpc_short}} uses a {{site.data.keyword.vpc_short}}-native Container Network Interface (CNI) plug-in with direct pod IP address assignment from {{site.data.keyword.vpc_short}} subnets. Classic clusters use Calico with overlay networking. {{site.data.keyword.vpc_short}}-native networking provides lesser latency and higher throughput for pod-to-pod communication.

Service load balancing
:   Clusters on {{site.data.keyword.vpc_short}} use {{site.data.keyword.vpc_short}} load balancers for Kubernetes `LoadBalancer` services. Classic clusters use classic load balancers. {{site.data.keyword.vpc_short}} load balancers provide better integration with {{site.data.keyword.vpc_short}} security groups and network ACLs.

Ingress performance
:   Ingress controllers on {{site.data.keyword.vpc_short}} clusters that are run as pods with {{site.data.keyword.vpc_short}} load balancer front ends. Performance is comparable to classic clusters but with better integration into {{site.data.keyword.vpc_short}} networking.

Storage performance
:   Clusters on {{site.data.keyword.vpc_short}} use {{site.data.keyword.block_storage_is_short}} for persistent volumes. Performance characteristics are similar to classic block storage, but {{site.data.keyword.vpc_short}} storage is zone-scoped.

### Pod-to-pod communication optimization
{: #c2v-performance-pod-to-pod-optimization}

Consider the following guidelines to optimize pod-to-pod communication:

- Colocate tightly coupled pods in the same zone by using pod affinity rules.
- Use Kubernetes services for stable endpoints rather than direct pod IP addresses.
- Configure appropriate resource requests and limits to prevent CPU throttling.
- Monitor network metrics by using {{site.data.keyword.mon_short}} to identify bottlenecks.

### Multizone cluster considerations
{: #c2v-performance-multizone-cluster}

Consider the following information for multizone clusters:

- Worker nodes in different zones communicate over the {{site.data.keyword.vpc_short}} backbone with approximately 2 ms of latency.
- Cross-zone pod communication incurs data transfer charges.
- Use topology-aware routing to prefer same-zone endpoints when possible.
- Deploy stateful workloads with zone-local storage to avoid cross-zone latency.

## Database performance considerations
{: #c2v-performance-database-performance}

{{site.data.keyword.vpc_short}} network characteristics can affect database performance.

### Connection pooling and network latency
{: #c2v-performance-connection-pooling}

Changes in network latency between the application tier and the database tier affect connection pool efficiency:

Connection pool sizing
:   Explicit routing in {{site.data.keyword.vpc_short}} adds 0.5-1 ms of latency that is compared with the implicit back-end network in classic infrastructure. Increase the connection pool size by 10-20% to maintain throughput under the same load.

Connection timeout tuning
:   Adjust connection timeout values to account for {{site.data.keyword.vpc_short}} network characteristics. Recommended minimums: 5 seconds for the connection timeout and 30 seconds for the query timeout.

Keepalive settings
:   Enable TCP keepalive with shorter intervals (60 seconds) to detect failed connections faster in the isolated network environment of {{site.data.keyword.vpc_short}}.

### Database replication performance
{: #c2v-performance-database-replication}

Cross-zone replication latency
:   Primary-replica replication across zones experiences approximately 2 ms of base latency. For synchronous replication, this latency directly affects write performance. Consider asynchronous replication for write-heavy workloads.

VPE impact on managed databases
:   {{site.data.keyword.databases-for}} services that you access through a VPE have lesser latency and higher throughput than public endpoint access. Configure VPEs before migration to maintain performance parity with classic private endpoints.

Replication monitoring
:   Monitor replication lag metrics closely during and after migration. Acceptable lag thresholds might need adjustment based on {{site.data.keyword.vpc_short}} network characteristics.

### Backup and restore performance
{: #c2v-performance-backup-restore-performance}

Consider the following information about backup and restore performance:

- Database backups to {{site.data.keyword.cos_short}} through a VPE achieve 2-5 times better throughput than public endpoint access.
- Restore operations from {{site.data.keyword.cos_short}} benefit similarly from VPE usage.
- Schedule backup windows during less-traffic periods to minimize performance impact.
- Test restore procedures in the {{site.data.keyword.vpc_short}} environment to validate performance before production cutover.

## Monitoring and observability overhead
{: #c2v-performance-monitoring-observability}

Monitoring and logging in {{site.data.keyword.vpc_short}} add performance overhead.

### VPC flow logs performance impact
{: #c2v-performance-flow-logs-impact}

{{site.data.keyword.vpc_short}} flow logs capture network traffic metadata for security and troubleshooting:

Performance overhead
:   Flow logs add minimal CPU overhead (less than 1%) but generate significant data volume. A busy instance can generate 1-10 GB of flow log data per day.

Storage costs
:   The flow log collector stores flow logs in {{site.data.keyword.cos_short}}. Configure lifecycle policies to archive or delete old logs to control costs.

Selective enablement
:   Enable flow logs only on subnets or instances that require detailed network analysis rather than you enabling them globally.

### Metrics collection frequency
{: #c2v-performance-metrics-collection}

{{site.data.keyword.mon_short}} agents collect system and application metrics:

- The default collection interval is 10 seconds.
- High-frequency collection (1 second) increases CPU overhead by 2-5%.
- Adjust the collection frequency based on monitoring requirements.
- Use metric aggregation to reduce data volume for long-term storage.

### Log shipping performance
{: #c2v-performance-log-shipping}

Consider the following guidelines for application and system logs that you ship to {{site.data.keyword.la_short}}:

- Use a VPE for {{site.data.keyword.la_short}} to avoid public internet latency.
- Configure log buffering to batch log entries and reduce network overhead.
- Filter logs at the source to reduce volume and network bandwidth consumption.
- Monitor log agent CPU and memory usage to prevent resource contention.

## Multiregion and disaster recovery performance
{: #c2v-performance-multiregion-dr}

Multiregion deployments and disaster recovery (DR) have specific performance considerations.

### Cross-region replication
{: #c2v-performance-cross-region-replication}

Consider the following information about data replication between {{site.data.keyword.vpc_short}}s in different regions:

Latency characteristics
:   Cross-region latency varies by geographic distance: US East to US South approximately 30 ms, US to Europe approximately 80-120 ms, and US to Asia approximately 150-250 ms.

Replication strategies
:   Use asynchronous replication for cross-region DR because of high latency. Synchronous replication across regions significantly affects write performance.

{{site.data.keyword.tg_short}} for multiregion
:   {{site.data.keyword.tg_short}} supports cross-region connections with per-GB data transfer charges. Evaluate costs against {{site.data.keyword.dl_short}} or VPN alternatives for high-volume replication.

### Failover time and recovery time objective
{: #c2v-performance-failover-rto}

The recovery time objective (RTO) in {{site.data.keyword.vpc_short}} differs from classic infrastructure:

DNS-based failover
:   Update DNS records to redirect traffic to the DR region. The DNS time to live (TTL) affects failover time (typically 60-300 seconds).

Global load balancer failover
:   The {{site.data.keyword.cis_short}} global load balancer provides automatic failover with health checks. Failover time is 30-60 seconds.

Application startup time
:   Factor in {{site.data.keyword.vpc_short}} instance boot time (2-5 minutes) and application initialization when you calculate the RTO.

### Global load balancing with {{site.data.keyword.cis_short}}
{: #c2v-performance-global-load-balancing}

{{site.data.keyword.cis_full_notm}} provides global load balancing across regions:

- Health check frequency is 15-60 seconds (configurable).
- Geographic routing reduces latency by directing users to the nearest region.
- Failover detection takes 30-90 seconds, depending on the health check configuration.
- {{site.data.keyword.cis_short}} integrates with {{site.data.keyword.vpc_short}} load balancers for multitier architectures.

## API and control plane performance
{: #c2v-performance-api-control-plane}

{{site.data.keyword.vpc_short}} API behavior and quotas can affect migration performance.

### VPC API rate limits
{: #c2v-performance-vpc-api-limits}

{{site.data.keyword.vpc_short}} APIs have rate limits to help ensure platform stability:

Default limits
:   Most {{site.data.keyword.vpc_short}} APIs allow 100 requests per minute per account. Some operations (for example, instance creation and volume attachment) have lesser limits.

Burst capacity
:   The API allows short bursts preceding the rate limit, but sustained high request rates result in HTTP `429` (Too Many Requests) responses.

Optimization strategies
:   Batch operations where possible, implement exponential backoff for retries, and cache API responses to reduce request frequency.

### Infrastructure as code performance
{: #c2v-performance-iac-performance}

Terraform&reg; and other infrastructure as code (IaC) tools provision {{site.data.keyword.vpc_short}} resources through the API:

Provisioning time
:   Large Terraform plans (more than 50 resources) can take 15-30 minutes to apply because of API rate limits and resource dependencies.

Parallel resource creation
:   Terraform creates independent resources in parallel (the default is 10 concurrent operations). Increase parallelism for faster provisioning, but monitor API rate limits.

State management
:   Store Terraform state in {{site.data.keyword.cos_short}} with state locking to prevent concurrent modifications and corruption.

### Resource quota impacts
{: #c2v-performance-resource-quotas}

{{site.data.keyword.vpc_short}} resource quotas affect deployment patterns:

- The default quotas are 25 {{site.data.keyword.vpc_short}}s per region, 100 instances per {{site.data.keyword.vpc_short}}, and 5 VPEs per {{site.data.keyword.vpc_short}}.
- If your current classic usage exceeds the {{site.data.keyword.vpc_short}} defaults, request quota increases before migration.
- Monitor quota usage to prevent provisioning failures during migration.
- Plan resource distribution across regions to avoid single-region quota limits.

## Cost-performance optimization
{: #c2v-performance-cost-performance-optimization}

Cost considerations affect performance decisions in {{site.data.keyword.vpc_short}}.

### Data transfer cost considerations
{: #c2v-performance-data-transfer-costs}

{{site.data.keyword.vpc_short}} data transfer pricing differs from classic infrastructure and affects architecture decisions:

Cross-zone transfer
:   Traffic between zones in the same region incurs per-GB charges. Minimize cross-zone traffic by colocating tightly coupled services.

Cross-{{site.data.keyword.vpc_short}} transfer
:   Traffic between {{site.data.keyword.vpc_short}}s through {{site.data.keyword.tg_short}} incurs per-GB charges plus {{site.data.keyword.tg_short}} connection fees.

Egress to internet
:   Outbound internet traffic through a public gateway or floating IP incurs per-GB charges. Use VPEs for {{site.data.keyword.cloud_notm}} service access to avoid egress charges.

### Reserved capacity and committed use
{: #c2v-performance-reserved-capacity}

{{site.data.keyword.vpc_short}} offers reserved capacity for predictable workloads:

- Terms of 1 year or 3 years provide up to a 30% discount compared with on-demand pricing.
- Reserved capacity is specific to a region and profile.
- Analyze classic usage patterns to identify candidates for reserved capacity.
- Combine reserved capacity with autoscaling for cost-effective performance.

### Right-sizing opportunities
{: #c2v-performance-right-sizing}

Migration to {{site.data.keyword.vpc_short}} provides an opportunity to optimize resource allocation:

- Performance testing in {{site.data.keyword.vpc_short}} might reveal overprovisioned classic instances.
- The fixed instance profiles of {{site.data.keyword.vpc_short}} encourage standardization and simplify capacity planning.
- Use {{site.data.keyword.mon_short}} to analyze resource utilization and identify right-sizing opportunities.
- Consider newer instance profile families (`bx2`, `cx2`, and `mx2`) for better price-performance.

## Application-level performance tuning
{: #c2v-performance-application-tuning}

Operating system and application settings can improve performance in {{site.data.keyword.vpc_short}}.

### TCP tuning for VPC
{: #c2v-performance-tcp-tuning}

The network architecture of {{site.data.keyword.vpc_short}} benefits from Transmission Control Protocol (TCP) parameter optimization:

TCP window scaling
:   Enable TCP window scaling to support high-bandwidth, high-latency connections. Recommended: `net.ipv4.tcp_window_scaling = 1`.

TCP buffer sizes
:   Increase TCP buffer sizes for better throughput. Recommended: `net.core.rmem_max = 134217728` and `net.core.wmem_max = 134217728`.

TCP congestion control
:   Use bottleneck bandwidth and round-trip propagation time (BBR) congestion control for improved throughput over {{site.data.keyword.vpc_short}} networks. Set `net.ipv4.tcp_congestion_control = bbr`.

### MTU considerations
{: #c2v-performance-mtu-considerations}

{{site.data.keyword.vpc_short}} supports jumbo frames for improved performance:

Default MTU
:   {{site.data.keyword.vpc_short}} instances use an MTU of 1500 by default, which is the same as classic infrastructure.

Jumbo frames
:   {{site.data.keyword.vpc_short}} supports an MTU of up to 9000 for traffic within the same {{site.data.keyword.vpc_short}}. Enable jumbo frames for storage and database replication traffic to reduce CPU overhead and improve throughput.

Path MTU discovery
:   Enable path MTU discovery (PMTUD) to automatically detect the optimal MTU for each connection. Set `net.ipv4.tcp_mtu_probing = 1`.

MTU configuration
:   Configure the MTU at the instance level by using network interface settings. Help ensure that all instances in the communication path support the same MTU to avoid fragmentation.

### Connection keepalive optimization
{: #c2v-performance-keepalive-optimization}

Optimized keepalive settings benefit the security groups and ACLs of {{site.data.keyword.vpc_short}}:

TCP keepalive
:   Enable TCP keepalive to detect failed connections faster. Recommended: `net.ipv4.tcp_keepalive_time = 60`, `net.ipv4.tcp_keepalive_intvl = 10`, and `net.ipv4.tcp_keepalive_probes = 6`.

Application-level keepalive
:   Configure application-level keepalive (HTTP and database) to complement TCP keepalive and maintain connection pool health.

Load balancer keepalive
:   Configure the {{site.data.keyword.vpc_short}} load balancer idle timeout to match the application keepalive settings. The default is 50 seconds for ALBs and 350 seconds for NLBs.
