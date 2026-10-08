---

copyright:
  years: 2026
lastupdated: "2026-10-08"

keywords: vpc firewall, firewall deployment, high availability, fortinet,
  palo alto, juniper, check point, f5, transit vpc, sdn connector

subcollection: classic-to-vpc
---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.cloud_notm}} VPC firewall options
{: #vpc-firewall-options}

{{site.data.keyword.vpc_full}} supports multiple firewall deployment options, including stand-alone, Active/Passive, and Active/Active high-availability configurations that you can deploy within a single availability zone or across multiple availability zones.
{: shortdesc}

{{site.data.keyword.vpc_short}} is a Layer 3 software-defined network (SDN) that provides flexible firewall deployment options to meet various security and availability requirements. Unlike {{site.data.keyword.cloud_notm}} Classic infrastructure, which uses a Layer 2 network architecture, VPC uses a Layer 3 SDN architecture that supports multiple firewall implementation patterns.

Before you select a deployment pattern, determine whether your workload requires a dedicated firewall appliance.

## What firewall options to consider when you migrate
{: #firewall-migration-considerations}

### Do you need a firewall in VPC?
{: #do-you-need-firewall}

VPC includes built-in security features that might be sufficient for some workloads:

Security groups (SGs)
:   Stateful firewalls that control traffic at the virtual server instance level. Security groups support allow rules only. When inbound traffic is allowed, return traffic is automatically allowed. Multiple security groups can be associated with a single instance. Security groups also support membership rules and references between security groups, which enable dynamic policy enforcement and simplified microsegmentation architectures.

Network Access Control Lists (NACLs)
:   Stateless firewalls that control traffic at the subnet level. NACLs support both allow and deny rules, and inbound and outbound rules must be explicitly defined. Rules are processed in sequence.

Public Gateway
:   Enables outbound internet access. Workloads cannot be reached from the internet unless a floating IP or public load balancer is attached.

Transit Gateway
:   Provides connectivity across VPCs, accounts, and regions without requiring a dedicated routing appliance. Depending on your workload requirements, other security controls might still be needed.

For more information about security groups and NACLs, see [Security in your VPC](/docs/vpc?topic=vpc-security-in-your-vpc).

### {{site.data.keyword.cloud_notm}} managed security services
{: #cloud-managed-security-services}

For specific security requirements, {{site.data.keyword.cloud_notm}} offers managed services that might eliminate the need for a dedicated firewall appliance:

* [{{site.data.keyword.cis_full}} (CIS)](https://www.ibm.com/products/cloud-internet-services){: external}: Provides Layer 7 (application layer) web application firewall (WAF), distributed denial-of-service (DDoS) protection, global load balancing, and content delivery network (CDN) capabilities. If you need only application-layer protection, CIS can meet your requirements because it is a fully managed service and does not require you to deploy a firewall appliance.
* [Virtual Private Network (VPN) for VPC and Client VPN services](https://www.ibm.com/products/vpn-for-vpc){: external}: These services provide site-to-site and client-to-site VPN connectivity without requiring a firewall appliance.

### When VPC native security might be sufficient
{: #when-vpc-native-security-sufficient}

VPC native security features might be sufficient in the following scenarios:

* Simple workload isolation requirements
* Basic ingress and egress traffic control at network and instance levels
* No advanced inspection or logging requirements
* Application-layer protection through {{site.data.keyword.cis_short}}
* VPN connectivity through VPC VPN service

### When to consider a dedicated firewall
{: #when-to-consider-dedicated-firewall}

Many enterprise and regulated workloads require capabilities beyond the native security features available in VPC. Common requirements and considerations that can influence the decision to deploy a dedicated firewall are described in the following sections.

These items are not a checklist. The presence of one or more requirements does not automatically indicate the need for a firewall appliance. Consider these factors alongside VPC native security features and managed services when you evaluate whether more controls are required.

#### Compliance and regulatory requirements
{: #compliance-requirements}

Some compliance frameworks and regulatory standards require security capabilities that extend beyond VPC native security controls.

PCI DSS, HIPAA, SOC 2
:   Many compliance frameworks mandate next-generation firewall (NGFW) capabilities.

Audit logging
:   Detailed traffic logs for integration with security information and event management (SIEM) systems such as IBM QRadar&reg;. VPC provides native logging capabilities, including [flow logs](/docs/vpc?topic=vpc-flow-logs) for network-level visibility and [data path logging for application load balancers](/docs/vpc?topic=vpc-datapath-logging). These services can support audit and monitoring requirements, depending on the level of detail and retention required.

Periodic security reports
:   Financial institutions and regulated industries often require comprehensive security reports every 6 to 12 months.

Simple Network Management Protocol (SNMP) traps and alerting
:   Integration with on-premises monitoring systems for real-time security event notification.

#### Advanced security capabilities
{: #advanced-security-capabilities}

A dedicated firewall can provide advanced traffic inspection, threat detection, and policy enforcement capabilities that extend beyond VPC native security features.

Intrusion Prevention System (IPS)
:   Uses deep packet inspection to detect and block malicious traffic patterns.

Intrusion Detection System (IDS)
:   Monitors network activity and alerts on suspicious behavior.

Application-layer inspection
:   Inspects traffic beyond Layer 4 (ports and protocols).

Layer 7 proxy and reverse proxy capabilities
:   Advanced firewall and proxy solutions can provide Hypertext Transfer Protocol (HTTP) and HTTP Secure (HTTPS) proxying, Transport Layer Security (TLS) termination, and deep application awareness.

Application Load Balancer (ALB) integration
:   ALBs can complement firewall deployments by providing Layer 7 traffic distribution and TLS termination before or alongside inspection architectures.

Server Name Indication (SNI) routing
:   Supports hostname-based routing and inspection policies for HTTPS applications.

Egress SNI filtering
:   Some vendor solutions support outbound HTTPS filtering based on SNI values, which enables policy enforcement for outbound internet access without full TLS decryption.

Command-and-control (C2) blocking
:   Automatically blocks connections to known malicious servers.

Data loss prevention (DLP)
:   Inspects and controls sensitive data that exits your network.

URL filtering
:   Controls access to websites by category or by specific URLs.

Quality of service (QoS)
:   Prioritizes critical application traffic.

#### Operational requirements
{: #operational-requirements}

Organizations with centralized security and governance requirements might benefit from a dedicated firewall deployment.

Centralized security policy management
:   Manages security rules across multiple VPCs from a single point.

Multi-tenant isolation
:   Provides separate security zones for different applications or customers.

Advanced logging and forensics
:   Provides detailed traffic logs with source and destination IP addresses, ports, and application identification.

Automation capabilities
:   Enables API-driven security policy updates and threat response.

Disaster recovery
:   Supports Active/Passive or Active/Active high availability across zones or regions.

#### Network architecture requirements
{: #network-architecture-requirements}

Certain network architectures require traffic inspection, segmentation, or routing patterns that are best implemented with a dedicated firewall.

Transit VPC (hub and spoke)
:   A centralized connectivity pattern for traffic between multiple VPCs. For more information, see [Transit VPC hub-and-spoke architecture](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc). This pattern can also serve as a centralized security inspection point when advanced controls are required.

Custom routing and traffic engineering
:   {{site.data.keyword.vpc_short}} supports custom routing tables and user-defined routes, which enables advanced traffic-steering patterns that are similar to {{site.data.keyword.cloud_notm}} Classic Gateway Appliance deployments, including Vyatta&reg;/Virtual Router Appliance (VRA), Juniper&reg; vSRX, and Fortinet&reg; vFSA solutions. Custom routes can direct traffic through firewalls, inspection points, transit gateways, or other network virtual appliances. This approach supports centralized security and segmented network architectures.

Hybrid cloud connectivity
:   Provides secure connections between VPC and on-premises data centers.

First-hop security
:   Stops and inspects all ingress and egress traffic at a single security checkpoint.

Zone-based security
:   Creates security zones (for example, DMZ, application tier, database tier) with controlled traffic flows between zones.

Cross-region connectivity and routing architecture
:   {{site.data.keyword.vpc_short}} uses a Layer 3 SDN model in which cross-region and cross-account connectivity is implemented by using the {{site.data.keyword.cloud_notm}} Transit Gateway service and VPN gateways, rather than firewall-centric routing. Transit Gateway provides the primary routing fabric between VPCs, while VPN gateways terminate encrypted Internet Protocol Security (IPsec) tunnels from on-premises or data center environments. In cross-region deployments, Transit Gateway instances are deployed in each region and connected through Transit Gateway peering to enable controlled inter-region routing without requiring firewall-based transit. This model replaces Classic infrastructure patterns where gateway firewalls were commonly used as centralized routing and connectivity hubs across data centers, accounts, or regions. For more information, see [Transit VPC hub-and-spoke architecture](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc).

### Classic infrastructure firewall context
{: #classic-firewall-context}

In {{site.data.keyword.cloud_notm}} Classic infrastructure, firewalls serve a critical role because of the Layer 2 network architecture:

Virtual LAN (VLAN) separation
:   Classic infrastructure uses VLANs to separate traffic. Transit VLANs connect gateways to public and private networks.

Gateway routing
:   VLANs associate resources and route traffic through gateway appliances (such as vFSA) for protection.

Untagged service networks
:   Transit VLANs act as service networks where packets are untagged.

Tagged workload networks
:   Automatic and Premium VLANs carry tagged traffic with subnets and IPs for workloads.

#### Key difference in VPC
{: #key-difference-vpc}

VPC uses a Layer 3 SDN architecture and the vendor-provided SDN Connector to provide flexible security options. However, the advanced security capabilities that firewalls provide remain essential for many enterprise workloads.

Unlike Classic infrastructure, which relies on VLAN-based routing through gateway appliances such as VRA (Vyatta), VPC uses routing tables, custom routes, and transit architectures to steer traffic through security and inspection services.

### Decision framework
{: #firewall-decision-framework}

Use the following framework to determine your firewall requirements:

1. Document your compliance, security, and operational requirements.
1. Determine whether security groups, NACLs, and public gateways meet your requirements.
1. Consider {{site.data.keyword.cloud_notm}} managed services:
   * For application-layer (Layer 7) protection only, use [{{site.data.keyword.cis_short}}](https://www.ibm.com/products/cloud-internet-services){: external}.
   * For VPN connectivity, use [VPC VPN service](https://www.ibm.com/products/vpn-for-vpc){: external}.
1. List capabilities that VPC native security and managed services cannot provide.
1. If a network firewall appliance is required, for most workloads start with the [Fortinet FortiGate VM PayGo offering](/docs/licensed-firewall), which includes the license and FortiCare Premium support. It is available in three topologies: Stand-alone, Active/Passive HA (Single Zone), and Active/Passive HA (Cross Zone). Choose the topology based on your availability and performance requirements.

   Before you select a firewall vendor, review the [vendor support matrix for high availability (HA) firewall deployments](#vendor-support-matrix-firewall-deployment-licensing).

   Vendor support varies by deployment mechanism, including Network Load Balancer (NLB), Border Gateway Protocol (BGP) over Generic Routing Encapsulation (GRE), SDN Connector-based failover, or bare metal virtualization. Not all vendors support all deployment patterns, and custom images and vendor-specific licensing configurations require validation.

Each firewall deployment pattern includes its characteristics, available solutions, and implementation options.

## Firewall offering types
{: #firewall-offering-types}

Firewall solutions in {{site.data.keyword.vpc_short}} are available through three primary offering models. Understanding these models can help you evaluate vendor support, deployment options, licensing responsibilities, and support boundaries. For most new deployments, the IBM-licensed Fortinet FortiGate VM PayGo offering is the recommended starting point because it includes the license and support and eliminates procurement and lifecycle management overhead.

### BYOA (Bring Your Own Appliance)
{: #byoa}

Bring Your Own Appliance (BYOA) deployments use customer-managed custom images instead of {{site.data.keyword.cloud_notm}} catalog offerings. You are responsible for obtaining and maintaining appliance licenses, software updates, and vendor support entitlements.

The following characteristics apply to BYOA deployments:

* You can deploy firewall vendors, software versions, and configurations that are not available through the {{site.data.keyword.cloud_notm}} catalog.
* You are responsible for validating high-availability configurations, automation workflows, routing integration, and vendor support.

For more information, see [Custom images, BYOA, and vendor support](#custom-images-byoa-vendor-support), and related deployment patterns, such as [Stand-alone deployments](#standalone-deployment) and [Active/Passive HA models](#active-passive-single-zone).

### Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)
{: #paygo}

The PayGo offering is an IBM-licensed Fortinet FortiGate firewall solution that you purchase directly through {{site.data.keyword.cloud_notm}}. The license and FortiCare Premium support are included with the offering and billed through {{site.data.keyword.cloud_notm}}. You do not need to separately procure, register, or renew a firewall license. Support for additional vendors is planned.

PayGo eliminates vendor procurement, license tracking, and renewal overhead. IBM automatically applies the license at provisioning time and manages it throughout the lifecycle. For most new Fortinet FortiGate deployments, PayGo is the recommended offering.

The following three Fortinet FortiGate VM PayGo catalog offerings are available, which cover all supported deployment topologies:

| Offering | Topology | Catalog link |
| -------- | -------- | ------------ |
| Fortinet FortiGate VM NGFW - Single (IBM reseller) | Stand-alone (Single VM) | [View in catalog](/catalog/content/ibm-fortigate-terraform-payg-6f8340d8-d6ef-420e-b50e-e305099917c6-global){: external} |
| Fortinet FortiGate VM NGFW - A/P HA (IBM reseller) | Active/Passive HA - Single Zone | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-terraform-payg-264eea02-7f0f-41b7-86f5-4adbb349430f-global){: external} |
| Fortinet FortiGate VM NGFW - Cross Zone A/P HA (IBM reseller)| Active/Passive HA - Cross Zone | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-CZ-terraform-payg-0d38cbcc-403a-430d-9a70-82221de0040b-global){: external} |
{: caption="FortiGate VM PayGo catalog offerings" caption-side="bottom"}

All three FortiGate PayGo offerings use the same deployment topologies as the BYOL offerings. IBM includes and manages the license and support. For details on license plans, deployment sizes, and feature entitlements, see [About firewall license plans and instance profiles](/docs/licensed-firewall?topic=licensed-firewall-about-firewall-license-plans-and-instance-profiles).

Review the following important considerations before you order a FortiGate VM PayGo offering:

Customer-managed service
:   IBM manages licensing and support coordination, but you are responsible for deploying, configuring, maintaining, and patching your FortiGate instances. For details, see [Shared responsibilities for FortiGate licensed firewall](/docs/licensed-firewall?topic=licensed-firewall-shared-responsibilities).

License plan is permanent
:   You cannot change the license plan after provisioning. If you need a different plan, you must redeploy.

Regional availability
:   All license plans are available in all supported regions.

For more information, see [Licensing models](#licensing-models) and [Deployment options](#deployment-options).

### BYOL catalog offering
{: #byol-catalog-offering}

Bring Your Own License (BYOL) catalog offerings are vendor-provided firewall images that are available through the {{site.data.keyword.cloud_notm}} catalog. You deploy from an approved catalog tile but provide your own firewall license directly from the vendor.

The following characteristics apply to BYOL catalog offerings:

* Vendor-provided images are available through the {{site.data.keyword.cloud_notm}} catalog.
* You obtain and manage firewall licenses directly from the vendor.
* Offerings often include vendor-tested deployment automation and reference architectures.
* Vendor support follows the vendor's licensing and support terms.

Examples include Juniper, Check Point&reg;, and other firewall offerings that are available through the {{site.data.keyword.cloud_notm}} catalog.

For more information, see [Licensing models](#licensing-models) and applicable deployment patterns, such as [Active/Active HA (Single Zone)](#active-active-single-zone) and [Active/Passive HA (Multizone)](#active-passive-multizone), which commonly use BYOL-based images.

## Comparison of firewall deployment options
{: #firewall-deployment-option-comparison}

The following table summarizes the most common firewall deployment options in VPC. Active/Passive deployments are the recommended pattern for enterprise workloads. Active/Active deployments that use NLBs or BGP over GRE are typically used for scalability, traffic distribution, and routing flexibility.

In BGP over GRE-based deployments, overall throughput depends on the firewall appliance architecture and GRE processing efficiency. In some implementations, GRE encapsulation and routing can reach central processing unit (CPU) capacity limits and restrict horizontal scaling.
{: note}

For detailed implementation guidance and reference architectures, see the [Transit VPC documentation](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc) and associated deployment guides.
{: tip}

| Feature | [Stand-alone](#standalone-deployment) | [Active/Active HA (Single Zone)](#active-active-single-zone) | [Active/Passive HA (Single Zone)](#active-passive-single-zone) | [Active/Passive HA (Multizone)](#active-passive-multizone) | [Active/Active HA (Multizone)](#active-active-multizone) |
| --------- | ------------------------------------- | --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | --------------------------------------------- |
| High availability/failover method | N/A | [Route mode network load balancer (RMNLB)](#route-mode-nlb-technical-details) or BGP over GRE | Virtual server instance: [SDN Connector](#sdn-connector-overview)[^sdn] \n Bare metal server: [Virtual network floating interface](#bare-metal-servers-reference) | Virtual server instance: [SDN Connector](#sdn-connector-overview)[^sdn] \n Bare metal server: [Virtual network floating interface](#bare-metal-servers-reference) | BGP over GRE or per-zone [RMNLB](#route-mode-nlb-technical-details) with optional state synchronization for asymmetric routing scenarios |
| Deployment complexity | Low | Medium | Medium | High | High |
| Compute options | Virtual server instance or [bare metal server](#bare-metal-servers-reference) | Virtual server instance | Virtual server instance or [bare metal server](#bare-metal-servers-reference) | Virtual server instance | Virtual server instance |
| Performance | [See performance factors](#performance-factors) | [See performance factors](#performance-factors) | [See performance factors](#performance-factors) | [See performance factors](#performance-factors) | [See performance factors](#performance-factors) |
| Public ingress support | Yes | RMNLB with public address range | Limited: VPC integration only and BYOA with bare metal | Limited:  Fortinet native VPC integration only | None |
| Supported vendors | Fortinet FortiGate VM PayGo, BYOA/BYOL | Fortinet FortiGate VM PayGo (Stand-alone), BYOA/BYOL using [RMNLB](#route-mode-nlb-technical-details) \n BGP-capable vendors that use BGP over GRE | Fortinet FortiGate VM PayGo, BYOA with [bare metal](#bare-metal-servers-reference) | Fortinet FortiGate VM PayGo, BYOA with [bare metal](#bare-metal-servers-reference) | Fortinet FortiGate VM PayGo (Stand-alone), BYOA/BYOL using BGP over GRE |
| BYOA / Vendor support qualification[^cv] | Customer validation required | Customer validation required | Customer validation required | Customer validation required | Customer validation required |
| Use case | Development and testing, small workloads | High throughput through scaling | Production (zone-level resilience) | Production (regional resilience) | High throughput + regional resilience |
| Licensing | BYOL or PayGo[^pg] | BYOL or PayGo (Stand-alone only)[^pg4] | BYOL or PayGo[^pg] | BYOL or PayGo[^pg] | BYOL or PayGo (Stand-alone only)[^pg4] |
{: caption="Comparison of firewall deployment patterns" caption-side="bottom"}
{: row-headers}
{: summary="This table has row and column headers. The row headers in the first column identify the feature. The column headers identify the deployment topology. To understand feature support for a given topology, navigate to the row for the feature and find the cell for the topology column."}

[^sdn]: Virtual Router Redundancy Protocol (VRRP), Pacemaker, or vendor-specific HA mechanisms might be supported depending on vendor and design.

[^cv]: Custom or BYOA vendor images might require customer-managed integration and validation. Vendor image availability does not necessarily imply {{site.data.keyword.vpc_short}} compatibility or vendor-supported operation. For more information, see [Custom images and vendor support](#custom-images-byoa-vendor-support).

[^pg]: FortiGate VM PayGo is available for Fortinet FortiGate deployments. Three offerings are available: Stand-alone, Active/Passive HA (Single Zone), and Active/Passive HA (Cross Zone). The license plan cannot be changed after deployment. For full details, see [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

[^pg4]: FortiGate VM PayGo is available for Fortinet FortiGate Stand-alone deployments only. To use a licensed firewall in this pattern, deploy a Stand-alone FortiGate VM PayGo instance and integrate it into the Active/Active design. The license plan cannot be changed after deployment. For full details, see [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

After you identify a deployment pattern, review the [vendor support matrix for HA firewall deployments](#vendor-support-matrix-firewall-deployment-licensing) to determine which firewall vendors support the selected high availability architecture, routing model, and automation mechanism.

Public ingress support indicates whether a deployment pattern provides native handling of inbound internet traffic and associated failover behavior for public endpoints.

Some patterns, such as Active/Active designs that use RMNLB or BGP over GRE, provide traffic forwarding or routing control but do not inherently provide ingress failover for public IP endpoints. In these cases, a separate ingress design is required. This design might include route-mode load balancing, public address range routing, or vendor-specific automation, such as SDN Connector integration for supported firewall appliances.

Bare metal-based deployments might also implement advanced ingress and failover behavior by using customer-managed virtualization layers. For example, Kernel-based Virtual Machine (KVM)/Quick Emulator (QEMU)-based virtualization, {{site.data.keyword.redhat_openshift_notm}} Virtualization, or VM mobility patterns similar to vMotion&reg;. In these architectures, the customer-managed platform manages instance-level mobility and failover behavior rather than the VPC networking layer.

BYOA deployments enable consistent Active/Passive designs across multiple vendors by enabling customer-controlled automation for failover and ingress handling. When combined with SDN Connector integration or custom automation, BYOA deployments can support standardized public ingress patterns for firewall appliances across both virtual server instances and bare metal deployments.

All deployment patterns in the table can be implemented within a single VPC. "Single VPC" is a deployment context rather than a separate firewall architecture, and each option supports it, depending on availability, scale, and routing requirements.

### Public ingress routing overview
{: #public-ingress-support}

For details on public ingress support across deployment models, see [Public ingress support considerations](#public-ingress-support-considerations).

## Vendor support matrix for HA firewall deployments in {{site.data.keyword.cloud_notm}} VPC
{: #vendor-support-matrix-firewall-deployment-licensing}

The following table summarizes vendor support for high availability (HA) firewall deployment patterns in {{site.data.keyword.vpc_short}}.

Vendor support varies by HA deployment pattern, automation requirements, routing model, and vendor-specific capabilities.

| Deployment topology | Fortinet | Juniper vSRX | Check Point | Palo Alto&reg; | Other vendors (custom images) |
| ------------------- | -------- | ------------ | ----------- | --------- | ----------------------------- |
| Active/Passive[^ap] | Supported \n (BYOL or PayGo; native SDN Connector integration) | BYOA | BYOA | BYOA | BYOA |
| Active/Passive bare metal deployment[^bmd] | BYOA | BYOA | BYOA | BYOA | BYOA |
| Active/Active (HA â€“ RMNLB) | Supported (BYOL) | Supported (BYOL) | Supported (BYOL) | BYOA | BYOA, if transparent routing is compatible |
| Active/Active (HA â€“ BGP over GRE) | Supported (BYOL) | Supported (BYOL) | Supported (BYOL, if BGP-capable) | BYOA, if BGP-capable | BYOA, if BGP and GRE are capable |
{: caption="Vendor support matrix for HA firewall deployments in {{site.data.keyword.vpc_short}}" caption-side="bottom"}
{: row-headers}
{: summary="This table has row and column headers. The row headers in the first column identify the deployment topology. The column headers identify the firewall vendor. To find vendor support for a specific topology, navigate to the topology row and find the cell for the vendor column."}

[^ap]: Active/Passive deployments include both single-zone and cross-zone (multizone) configurations. Fortinet provides native SDN Connector integration for automated cross-zone failover, while other vendors require customer-managed automation or HA mechanisms such as VRRP or Pacemaker.

[^bmd]: Bare metal deployment is a deployment platform rather than an HA architecture. HA behavior on bare metal depends on the selected clustering, failover, or virtualization technology (for example, virtual network floating interfaces, VRRP, Pacemaker, vendor-specific HA mechanisms, or customer-managed virtualization platforms).

Catalog image availability does not imply support for all deployment patterns. Vendor-specific automation, routing capabilities, and HA capabilities vary by vendor and deployment architecture.
{: note}

## Common deployment patterns
{: #common-deployment-patterns}

Firewall deployments in VPC use established network architecture patterns to control, inspect, and secure traffic flows.

### Transit VPC (hub and spoke)
{: #transit-vpc-pattern}

A Transit VPC (hub-and-spoke) architecture centralizes network security and traffic inspection. Firewall appliances are deployed in a dedicated transit VPC that serves as a hub for connectivity between enterprise networks, the internet, {{site.data.keyword.cloud_notm}} workloads, and other connected platforms.

Traffic between spoke VPCs, on-premises environments, and external networks is routed through the transit VPC for inspection and policy enforcement. East-west traffic between connected environments can also be controlled through centralized routing and firewall policies.

This pattern provides centralized security controls, consistent policy enforcement, and simplified management for environments that contain multiple VPCs or hybrid-cloud connectivity requirements.

![A central transit VPC connected to multiple spoke VPCs, with firewall appliances in the hub inspecting traffic flows between spoke VPCs, on-premises environments, and the internet](images/firewall-transit-vpc.svg){: caption="Transit VPC hub-and-spoke architecture with centralized firewall inspection" caption-side="bottom"}

For more information, see [Securing multiple landing zones with a transit VPC and advanced security capabilities](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc).

### Single VPC
{: #single-vpc-pattern}

A single VPC architecture deploys firewall appliances directly within the same VPC as the protected workloads. VPC routing tables and firewall policies control traffic flows within that VPC.

A single VPC deployment is suitable for isolated workloads, smaller environments, or deployments that do not require centralized inspection across multiple VPCs. It provides a less complex architecture with fewer networking components while it still enables north-south traffic inspection and workload segmentation.

For larger environments or deployments that require centralized security controls across multiple VPCs, a Transit VPC architecture is typically preferred.

![Firewall appliances deployed within a single VPC, with VPC routing tables directing traffic through the firewall for inspection and workload segmentation](images/firewall-single-vpc.svg){: caption="Single VPC deployment with firewall appliances and routing table-based traffic inspection" caption-side="bottom"}

## Deployment options
{: #deployment-options}

{{site.data.keyword.vpc_short}} supports multiple deployment options to address different availability and scalability requirements.

Vendor support differs by deployment topology and implementation model. Before you select a firewall vendor, review the [vendor support matrix for HA firewall deployments](#vendor-support-matrix-firewall-deployment-licensing).

For most enterprise production workloads, virtual server instance-based Active/Passive HA deployments are the recommended starting point for {{site.data.keyword.vpc_short}} firewall architectures.

This deployment model provides:

* High availability with lower operational complexity
* Compatibility with vendor-supported automation patterns
* Flexible scaling and hourly billing
* Routing that is less complex than Active/Active BGP-based routing architectures
* Easier migration from {{site.data.keyword.cloud_notm}} Classic gateway deployments

More advanced architectures, such as Active/Active multizone deployments by using BGP over GRE, are typically selected for large-scale throughput, advanced traffic engineering, or specialized routing requirements.

### Stand-alone deployment
{: #standalone-deployment}

A stand-alone deployment consists of a single firewall instance that is deployed from a single-VM catalog tile, and is suitable for proof-of-concept or noncritical workloads.

The stand-alone catalog tile is also the per-instance building block for [Active/Active HA (single zone)](#active-active-single-zone) and [Active/Active HA (multizone)](#active-active-multizone) topologies. In those patterns, you deploy the stand-alone tile two or three times and connect the instances by using an RMNLB or BGP over GRE. A multi-instance deployment that uses one of these HA patterns is suitable for production workloads.

#### Characteristics
{: #standalone-characteristics}

* Least complex deployment model when deployed as a single instance
* No automatic failover when deployed alone
* Lower cost for single-instance deployments
* Any vendor supported
* Serves as the per-instance unit for Active/Active single-zone and multizone HA topologies when deployed multiple times with RMNLB or BGP over GRE

#### Available solutions
{: #standalone-available-solutions}

The following table outlines the available stand-alone firewall solutions from leading vendors, along with their corresponding products and catalog links. Vendor compatibility depends on the selected deployment mechanism and might require validation for BYOL and custom image deployments.

| Vendor | Product | Licensing | Catalog link |
| -------- | --------- | --------- | -------------- |
| Fortinet | FortiGate VM NGFW - Single (recommended)[^pgn] | PayGo (license and support included) | [View in catalog](/catalog/content/ibm-fortigate-terraform-payg-6f8340d8-d6ef-420e-b50e-e305099917c6-global){: external} |
| Fortinet | FortiGate NGFW - Single VM | BYOL | [View in catalog](/catalog/content/ibm-fortigate-terraform-deploy-1f878ca9-069f-42ca-9ed9-5b461d4d5231-global){: external} |
| Juniper | Next-Gen SASE Firewall - BYOL | BYOL | [View in catalog](/catalog/content/jnpr-nextgen-fw-vsrx-74b4b3ba-2a05-460d-afba-98e4d012f53a-global?catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2Fc2VhcmNoPXZtLXNlcmllcyUyNTIwZmlyZXdhbGwlMjUyMGJ5b2wjc2VhcmNoX3Jlc3VsdHM%3D){: external} |
| Check Point | CloudGuard&reg; Network Security Firewall | BYOL | [View in catalog](/catalog/content/check-point-cloudguard-network-security-firewall-with-threat-prevention-1f1f50fe-e41d-4715-9ba6-02d37d76596c-global?catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2Fc2VhcmNoPWNoZWNrJTI1MjBwb2ludCNzZWFyY2hfcmVzdWx0cw%3D%3D){: external} |
| F5&reg; | BIG-IP&reg; Virtual Edition for VPC | BYOL | [View in catalog](/catalog/content/ibmcloud_schematics_bigip_multinic_declared-1.0-d33f1544-e938-478a-b0dd-d883370f08d0-global?catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2Fc2VhcmNoPUY1I3NlYXJjaF9yZXN1bHRz){: external} |
{: caption="Available stand-alone firewall solutions" caption-side="bottom"}

[^pgn]: FortiGate VM PayGo includes the license and FortiCare Premium support. The license plan cannot be changed after deployment. This is a customer-managed service. See [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

#### Best for
{: #standalone-best-for}

* Development and testing environments
* Noncritical workloads
* Cost-sensitive deployments
* Proof-of-concept projects
* Multi-instance Active/Active HA architectures when combined with RMNLB or BGP over GRE

A single, unconnected stand-alone instance does not provide firewall high availability and is not suitable for mission-critical production workloads. However, if you deploy multiple stand-alone instances and connect them by using RMNLB or BGP over GRE, the result is a production-grade Active/Active HA topology. See [Active/Active HA (single zone)](#active-active-single-zone) and [Active/Active HA (multizone)](#active-active-multizone).
{: note}

### Active/Active HA (single zone)
{: #active-active-single-zone}

Multiple firewall instances actively process traffic within a single availability zone by using either an RMNLB or BGP over GRE for dynamic routing.

#### Characteristics
{: #active-active-sz-characteristics}

* Load balancing across multiple firewall instances
* Scalable traffic distribution capacity
* Supports RMNLB or BGP over GRE
* Vendor support depends on the deployment mechanism
* Single-zone high availability

#### Available solutions
{: #active-active-sz-available-solutions}

The Active/Active HA (single zone) topology is supported through BYOL and BYOA deployments only. No PayGo offering is available for this topology. Vendor compatibility depends on the selected deployment mechanism (RMNLB or BGP over GRE) and requires validation for BYOL and custom-image deployments.

| Vendor | Product | Catalog link |
| -------- | --------- | -------------- |
| Fortinet | FortiGate NGFW (BYOL) | [View Fortinet FortiGate in catalog](/catalog/content/ibm-fortigate-terraform-deploy-1f878ca9-069f-42ca-9ed9-5b461d4d5231-global){: external} |
| Juniper | vSRX Firewall (BYOL) | [View Juniper vSRX Firewall in catalog](/catalog/content/jnpr-nextgen-fw-vsrx-74b4b3ba-2a05-460d-afba-98e4d012f53a-global){: external} |
| Check Point | CloudGuard Network Security Firewall (BYOL) | [View Check Point CloudGuard in catalog](/catalog/content/check-point-cloudguard-network-security-firewall-with-threat-prevention-1f1f50fe-e41d-4715-9ba6-02d37d76596c-global){: external} |
| F5 | BIG-IP Virtual Edition for VPC (BYOL) | [View F5 BIG-IP Virtual Edition in catalog](/catalog/content/ibmcloud_schematics_bigip_multinic_declared-1.0-d33f1544-e938-478a-b0dd-d883370f08d0-global){: external} |
{: caption="Available Active/Active HA (single zone) solutions" caption-side="bottom"}

#### Implementation
{: #active-active-sz-implementation}

* The firewall must support the BGP routing protocol for BGP over GRE deployments.
* RMNLB requires IP spoofing support on firewall instances.
* Vendor capability determines eligibility for BGP-based deployments.
* Some vendors might require custom configuration or validation, depending on the routing mode.

#### Best for
{: #active-active-sz-best-for}

* High-throughput single-zone deployments
* Scalable traffic distribution architectures
* Environments that use RMNLB or BGP over GRE
* Firewall scaling within a single availability zone

### Active/Passive HA (single zone)
{: #active-passive-single-zone}

An Active/Passive HA (single zone) deployment consists of two firewall instances in an Active/Passive configuration within a single availability zone. The passive instance takes over if the active instance fails.

Active/Passive HA (single zone) provides zone-level redundancy for production workloads.

#### Characteristics
{: #active-passive-sz-characteristics}

* Zone-level high availability
* Automatic failover that uses SDN Connector
* Supports virtual server instance and bare metal server deployments
* Tested with Fortinet and Palo Alto

#### Available solutions
{: #active-passive-sz-available-solutions}

The following table lists the available Active/Passive HA (single zone) firewall solutions.

| Vendor | Product | Licensing | Catalog link |
| -------- | --------- | --------- | -------------- |
| Fortinet | FortiGate-VM NGFW Firewall - A/P HA (recommended)[^pgn2] | PayGo (license and support included) | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-CZ-terraform-payg-0d38cbcc-403a-430d-9a70-82221de0040b-global){: external} |
| Fortinet | FortiGate NGFW - A/P HA | BYOL | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-terraform-deploy-5dd3e4ba-c94b-43ab-b416-c1c313479cec-global?catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2Fc2VhcmNoPUZvcnRpZ2F0ZSNzZWFyY2hfcmVzdWx0cw%3D%3D){: external} |
{: caption="Available Active/Passive HA (single zone) solutions" caption-side="bottom"}

[^pgn2]: FortiGate VM PayGo includes the license and FortiCare Premium support. The license plan cannot be changed after deployment. This is a customer-managed service. See [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

#### Implementation
{: #active-passive-sz-implementation}

* Virtual server instance deployments use SDN Connector for automatic failover.
* Bare metal deployments use virtual network floating interfaces for failover.
* Vendor capability determines automation support for failover orchestration.
* Some vendors might require custom automation or manual failover, depending on the deployment model.

#### Best for
{: #active-passive-sz-best-for}

* Production workloads that require zone-level resilience
* Applications that can tolerate zone-level outages
* Standard enterprise firewall high availability deployments

For Fortinet FortiGate virtual server instance deployment guidance, see [Transit VPC with FortiGate Solution Guide](/docs/pattern-transit-vpc-fortigate) and [FortiGate with SDN Connector](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc#fortigate-with-SDN-connector).

### Active/Passive HA (multizone)
{: #active-passive-multizone}

An Active/Passive HA (multizone) deployment consists of two firewall instances in an Active/Passive configuration across multiple availability zones, and provides regional-level high availability.

#### Characteristics
{: #active-passive-mz-characteristics}

* Regional high availability
* Protection against zone failures
* Automatic failover across zones
* SDN Connector-based automation for virtual server instance deployments
* Bare metal deployments use virtual network floating interfaces or customer-managed failover mechanisms
* Currently, optimized for Fortinet

In GRE-based designs, throughput characteristics depend on the firewall appliance processing architecture and might not scale linearly with additional instances.

#### Available solutions
{: #active-passive-mz-available-solutions}

The following table lists the available Active/Passive HA (multizone) firewall solutions.

| Vendor | Product | Licensing | Catalog link |
| -------- | --------- | --------- | -------------- |
| Fortinet | FortiGate-VM NGFW - Cross Zone A/P HA (recommended)[^pgn3] | PayGo (license and support included) | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-CZ-terraform-payg-0d38cbcc-403a-430d-9a70-82221de0040b-global){: external} |
| Fortinet | FortiGate NGFW - Cross Zone HA | BYOL | [View in catalog](/catalog/content/ibm-fortigate-AP-HA-terraform-deploy-5dd3e4ba-c94b-43ab-b416-c1c313479cec-global?catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2Fc2VhcmNoPUZvcnRpZ2F0ZSNzZWFyY2hfcmVzdWx0cw%3D%3D){: external} |
{: caption="Available Active/Passive HA (multizone) solutions" caption-side="bottom"}

[^pgn3]: FortiGate VM PayGo includes the license and FortiCare Premium support. The license plan cannot be changed after deployment. This is a customer-managed service. See [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

#### Implementation
{: #active-passive-mz-implementation}

* Virtual server instance deployments use SDN Connector for cross-zone failover orchestration.
* Bare metal deployments require customer-managed failover mechanisms (for example, VRRP, Pacemaker, or equivalent vendor HA tools).
* Failover between zones depends on routing updates and zone-aware VPC routing behavior.
* Vendor automation support varies and requires validation for BYOL and custom images.

#### Best for
{: #active-passive-mz-best-for}

* Mission-critical production workloads
* Applications that require maximum availability
* Compliance requirements for regional resilience
* Disaster recovery scenarios

#### Failover method
{: #active-passive-mz-failover-method}

SDN Connector with cross-zone awareness automatically updates routing when a zone failure occurs.

### Active/Active HA (multizone)
{: #active-active-multizone}

Multiple firewall instances actively process traffic across multiple availability zones by using BGP over GRE for routing scalability and resiliency.

#### Characteristics
{: #active-active-mz-characteristics}

* Regional high availability with load balancing; route-mode designs require a per-zone NLB.
* Highest throughput and resilience
* BGP support is required
* Most complex deployment model

#### Implementation
{: #active-active-mz-implementation}

* The firewall must support the BGP routing protocol.
* Any vendor with BGP capability is supported for BGP-based deployments.
* For more information, see [Virtual firewall appliances with BGP Over GRE](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc#Virtual-firewall-Appliances-with-BGP-over-GRE).

Stateful inspection in Active/Active designs requires symmetric routing through the same firewall instance. When asymmetric routing can occur, some firewall vendors (including Fortinet FortiGate and supported Palo Alto Networks deployments) use state synchronization mechanisms. Examples include Fortinet FortiGate Session Life Support Protocol (FGSP) and equivalent session synchronization features that maintain session continuity across instances.

#### Best for
{: #active-active-mz-best-for}

* Enterprise-scale deployments
* Maximum throughput and availability requirements
* Complex routing scenarios
* Multi-region architectures

BGP over GRE tunnels provide dynamic routing and automatic failover across availability zones.

#### Active/Active HA (multizone) with RMNLB and Fortinet state synchronization
{: #active-active-multizone-nlb-fgsp}

An Active/Active multizone deployment pattern is available that does not require BGP for routing control. RMNLBs, typically one per availability zone, are combined with state synchronization between firewall instances when flow recovery across zones is required.

In this design, firewall instances operate in Active/Active mode across multiple availability zones, and RMNLBs distribute traffic across the firewall pool while they preserve the source and destination IP addresses. Transparent inspection and stateful traffic handling are enabled without requiring dynamic routing protocols, such as BGP.

Fortinet deployments can use FGSP for session state synchronization between instances. FGSP helps maintain connection state consistency across availability zones when traffic for a session might be processed by different firewall instances.

FGSP is required only for asymmetric traffic scenarios or when session recovery is required following a zone failure. Symmetric traffic flows do not require FGSP.
{: note}

Active/Active multizone with RMNLB is useful in environments where BGP-based routing is not required or not supported, while you still require horizontal scaling and multizone availability.

Active/Active multizone with RMNLB integrates with the {{site.data.keyword.vpc_short}} routing model, where routing tables direct traffic toward the RMNLB as the next hop. The RMNLB then distributes traffic to available firewall instances for inspection and forwarding.

For cross-region architectures, you can combine this Active/Active model with Transit Gateway peering to extend inspection capabilities across regions while you maintain regional routing control boundaries.

## Supporting architecture concepts
{: #supporting-arch-concepts}

{{site.data.keyword.vpc_short}} firewall deployment options rely on underlying networking and automation capabilities that enable connectivity, high availability, routing control, and traffic inspection behavior across different architectures.

The foundational components and mechanisms are not deployment choices themselves, but building blocks used across multiple deployment patterns. Understanding them clarifies how traffic is routed, how failover is handled, and how different high availability models are implemented within {{site.data.keyword.vpc_short}}.

### Public ingress support considerations
{: #public-ingress-support-considerations}

You can direct public ingress traffic to firewall instances by using {{site.data.keyword.vpc_short}} public ingress capabilities and routing constructs.

The following considerations can help you evaluate public ingress support for different deployment models:

* Active/Passive deployments: Public ingress can be associated with public address ranges or floating IPs. VPC routing tables can then direct traffic to the active firewall instance for inspection and forwarding.
* Active/Active deployments with RMNLBs: Public ingress can be associated with public address ranges or other supported ingress constructs and routed to an RMNLB. The RMNLB then distributes traffic across available firewall instances for inspection.
* Bare metal deployments: Public ingress can be directed to firewall instances by using the same VPC routing capabilities, with implementation details depending on the deployment architecture.
* Other firewall vendors and custom images: Public ingress routing follows the standard {{site.data.keyword.vpc_short}} routing model. You must validate vendor-specific deployment and routing requirements.

For more information, see [About routing tables and routes](/docs/vpc?topic=vpc-about-custom-routes#routes-ingress).

### SDN Connector overview
{: #sdn-connector-overview}

The SDN Connector is a vendor-provided plug-in that enables automatic failover in Active/Passive HA configurations for virtual server instance-based deployments. It monitors the firewall cluster and automatically updates VPC routing tables when a failover occurs.

You can also implement similar functions by using customer-managed plug-ins, such as VRRP, vendor-specific heartbeat mechanisms, or automation frameworks (for example, Pacemaker), depending on the firewall vendor and deployment model.

Bare metal deployments do not use SDN Connector. Instead, they use virtual network floating interfaces.
{: important}

#### Vendor support (virtual server instance only)
{: #sdn-connector-vendor-support}

The following vendor support options are available for SDN Connector integration:

Fortinet FortiGate
:   Native SDN Connector integration is included in {{site.data.keyword.vpc_short}} images.

Other vendors
:   Must implement custom automation or use manual failover processes to support HA configurations.

For support information across all HA deployment models, see the [vendor support matrix for HA firewall deployments](#vendor-support-matrix-firewall-deployment-licensing).

#### How it works
{: #sdn-connector-how-it-works}

1. The SDN Connector continuously monitors the HA cluster state.
1. The SDN Connector detects when an active node changes.
1. The SDN Connector uses {{site.data.keyword.vpc_short}} APIs to update routing.
1. The SDN Connector redirects traffic to the new active node.

#### Failover process
{: #sdn-connector-failover-process}

The following sequence describes a typical failover event:

1. The active node fails or becomes unavailable.
1. The passive node detects the failure and becomes active.
1. The SDN Connector detects the cluster state change.
1. The connector queries VPC routing tables for routes that contain the old active IP address.
1. The connector deletes each old route.
1. The connector creates new routes that point to the new active node.
1. Traffic automatically flows through the new active node.

#### Benefits
{: #sdn-connector-benefits}

The SDN Connector provides the following benefits:

* Automatic failover without manual intervention
* Recovery that typically completes within seconds
* No external monitoring required
* Integrated with firewall HA mechanisms (Fortinet only)

### Bare metal servers
{: #bare-metal-servers-reference}

Bare metal servers provide dedicated hardware resources but require manual configuration and management.

Bare metal deployments can also be used as a customer-managed virtualization host layer for firewall workloads. In these designs, firewall instances run on a customer-managed hypervisor or orchestration platform, such as KVM/QEMU-based virtualization, {{site.data.keyword.redhat_openshift_notm}} Virtualization, or VM mobility frameworks similar to vMotion. Running firewall instances on a customer-managed platform enables workload-level mobility and controlled restart or relocation of firewall instances across physical hosts.

In these architectures, {{site.data.keyword.vpc_short}} networking does not directly provide high availability and ingress behavior. Instead, the customer-managed virtualization or orchestration layer handles failover, instance placement, and traffic continuity, in combination with network constructs such as virtual network floating interfaces, routing updates, or external automation tools.

Failover method
:   The following failover mechanisms are commonly used in bare metal deployments:

* Bare metal deployments use virtual network floating interfaces instead of SDN Connector for primary failover. In more advanced deployments, failover and instance mobility can also be implemented by using customer-managed virtualization platforms, such as KVM/QEMU-based orchestration or {{site.data.keyword.redhat_openshift_notm}} Virtualization. These platforms can support workload mobility or restart-based recovery across bare metal hosts, depending on the customer's architecture and configuration.
* Customer-managed virtualization platforms, such as KVM/QEMU-based orchestration or {{site.data.keyword.redhat_openshift_notm}} Virtualization, can provide workload mobility or restart-based recovery across bare metal hosts.
* Some environments implement live migration capabilities similar to VMware&reg; vMotion, where supported by the underlying virtualization platform. These solutions commonly rely on floating VLANs or equivalent Layer 2 network mobility mechanisms and are therefore typically limited to hosts connected to the same subnet within a zone.
* Vendor-specific clustering or heartbeat mechanisms, such as VRRP, Pacemaker, or firewall-native HA protocols, can also be used depending on the firewall appliance and deployment model.

Important limitations
:   Consider the following limitations when you use bare metal deployments:

* Manual configuration required: You must manually configure and manage the hypervisor and all virtual machines.
* Limited flexibility: Bare metal deployments cannot scale out as easily as virtual server deployments.
* Customer managed: You are responsible for the operating system and all software.
* Complexity: Expertise in hypervisor management and virtual machine configuration is required.
* Floating interface scope: Virtual network floating interfaces can move only within the same subnet. Because VPC subnets are zonal constructs, floating interface-based failover and mobility solutions that depend on floating VLANs or Layer 2 network mobility cannot be used for cross-zone failover.

Technical details
:   Key technical considerations include:

* The SDN Connector is not used for bare metal failover. Instead, failover is typically implemented through virtual network floating interfaces or customer-managed mechanisms. Therefore, virtual network floating interfaces are limited to movement within the same subnet and support failover only within a single zone.
* Tested vendors: Fortinet (PCI pass-through and `macvtap`) and Palo Alto (`macvtap`).

For more information, see [Virtual firewalls on VPC Bare Metal servers](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc#Virtual-firewall-Appliances-on-VPC-Bare-Metals).

Virtual server instance deployments are recommended for most use cases because of flexibility, ease of management, and hourly billing.
{: tip}

#### Best for
{: #bare-metal-best-for}

* Production workloads that require zone-level resilience
* Applications that can tolerate zone-level outages

### Cross-zone failover technical details
{: #cross-zone-failover-tech-details}

Cross-zone failover is more complex than single-zone failover because of zone-specific routing requirements and the need to update zone bindings.

#### Vendor support
{: #cross-zone-vendor-support}

The following vendor support options are available for cross-zone failover:

Fortinet FortiGate
:   Native SDN Connector with cross-zone support and automatic public address range zone binding updates.

Other vendors
:   Require custom automation to support cross-zone failover.

#### Failover process
{: #cross-zone-failover-process}

The following sequence describes a typical cross-zone failover event:

1. The Fortinet SDN Connector detects a zone failure or active node change.
1. The SDN Connector identifies and updates routes that point to the previous active node.
1. The SDN Connector updates the route zone to match the new active node's zone.
1. If you use Public Address Ranges for VPC, the public address range zone binding is updated for Fortinet deployments. Public address ranges support public IPs across multiple zones.

#### Route update pattern
{: #cross-zone-route-update-pattern}

For each route that points to an old active node:

1. `DELETE` the old route.
1. `CREATE` a new route with:

   * Same destination Classless Inter-Domain Routing (CIDR)
   * New `next_hop` IP (new active node)
   * New zone (new active node's zone)

#### Key points
{: #cross-zone-key-points}

Consider the following characteristics of cross-zone failover:

* Failover typically completes within seconds.
* A brief traffic interruption occurs during route updates.
* Automatic recovery without manual intervention (Fortinet only).
* VPC routing table routes cannot be updated with a new zone through `PATCH`. You must use `DELETE` and then `CREATE`.
* Floating IPs cannot move across zones (VPC limitation). Floating IPs support public IP access within a single zone.

   Floating IPs are zone-scoped resources, while Public Address Ranges are VPC-scoped and can be routed across zones by using routing tables.
   {: note}

### Public address range integration
{: #par-integration}

Public address ranges enable public-facing applications to preserve source IP addresses without Network Address Translation (NAT), while traffic is still routed through firewalls for inspection and security.

All firewall deployment patterns can support both private and public traffic flows. Public Address Ranges for VPC provide additional capabilities, such as source IP preservation and automated zone-binding updates for supported integrations.

#### What are public address ranges for VPC?
{: #what-is-par}

Public Address Ranges for VPC provide the following capabilities:

* A public subnet that is bound to a specific VPC and zone.
* Integration with routing tables that use the public internet as the traffic source.
* Source IP preservation without NAT by the VPC infrastructure.
* Optional NAT processing by the firewall for back-end applications.

#### Vendor support
{: #par-vendor-support}

The following vendor support options are available:

Fortinet FortiGate
:   Native Public Address Range integration with automatic zone-binding updates during cross-zone failover (FortiOS&reg; `7.6.3` and later).

Other vendors
:   Public address ranges are supported, but custom automation is required for zone-binding updates.

#### Use cases
{: #par-use-cases}

Public address ranges are commonly used for the following scenarios:

* Public-facing web applications that require source IP visibility
* Services with IP-based access control or logging requirements
* Compliance requirements for IP address logging
* DDoS protection deployments that require source IP preservation

#### How public address ranges work with firewalls
{: #par-how-it-works}

The following sequence describes a typical traffic flow:

1. The public address range is created and bound to a VPC and specific zone.
1. The routing table is configured with the public address range CIDR as the destination.
1. Traffic from the public internet matches the public address range routes.
1. Routes direct traffic through the firewall as the next hop.
1. The firewall inspects and forwards traffic to back-end applications.
1. Source IP addresses are preserved throughout the traffic flow.

#### Cross-zone failover with public address ranges (Fortinet only)
{: #par-cross-zone-failover}

When an active firewall node moves to a different zone, the Fortinet SDN Connector automatically runs the following actions:

1. The SDN Connector updates routing table routes, including `next_hop` and zone information.
1. The SDN Connector updates the public address range zone binding to match the new active node's zone.
1. The SDN Connector maintains traffic flow through the new active node.

#### Benefits
{: #par-benefits}

Public address ranges provide the following benefits:

* Source IP preservation for security and compliance requirements
* Transparent firewall operation for public traffic
* Automatic failover with public address range zone binding updates for supported Fortinet deployments
* No application changes required

### RMNLB technical details
{: #route-mode-nlb-technical-details}

Route mode is a feature of a network load balancer (NLB) that enables transparent firewall deployments in Active/Active configurations.

For Layer 7 use cases, application load balancers (ALBs) can also be integrated with firewall architectures to provide HTTP/HTTPS routing, TLS termination, and hostname-based application segmentation. ALBs are complementary to NLB route mode deployments, which operate at Layer 4.

#### How route mode works
{: #rmnlb-how-it-works}

Route mode provides the following capabilities:

* Preserves source and destination IP addresses without NAT
* Acts as a bump-in-the-wire for traffic inspection
* Distributes traffic across multiple firewall instances
* Maintains session affinity for stateful inspection

Stateful firewalls require symmetric traffic flows. Return traffic must traverse the same firewall instance that processed the original session. Design routing tables, load-balancing behavior, and failover mechanisms to avoid asymmetric routing conditions that can disrupt stateful inspection.

In Active/Active multizone deployments, you can combine RMNLB behavior with firewall-level state synchronization mechanisms such as Fortinet FGSP to maintain session consistency across multiple active instances. This approach allows scaling across zones without requiring BGP-based routing protocols while you preserve the symmetric traffic flows that stateful inspection requires.

#### Traffic flow
{: #rmnlb-traffic-flow}

Route mode behavior follows this traffic flow:

```text
REQUEST:  Client â†’ VPC Routing Table â†’ NLB (Route Mode) â†’ Firewall â†’ VPC Routing Table â†’ Server
RESPONSE: Server â†’ VPC Routing Table â†’ NLB (Route Mode) â†’ Same Firewall â†’ VPC Routing Table â†’ Client
```
{: pre}

#### Key characteristics
{: #rmnlb-key-characteristics}

Route mode deployments provide the following characteristics:

Transparent operation
:   Client and server see each other's real IPs.

No NAT
:   Source and destination IPs are preserved throughout the flow.

Session persistence
:   Return traffic uses the same firewall instance.

Load distribution
:   The NLB distributes traffic across active firewall instances.

Ingress behavior limitation
:   RMNLB provides traffic steering and distribution but does not provide public IP failover or endpoint mobility. Public ingress failover must be implemented through routing design or vendor-specific mechanisms.

Vendor-agnostic
:   Works with any firewall vendor.

#### Configuration requirements
{: #rmnlb-configuration-requirements}

The following configuration requirements apply to route mode deployments:

* NLB with route mode enabled
* IP spoofing enabled on firewall interfaces
* Security group rules configured to allow traffic
* Routing tables configured to use NLB as the next hop
* Firewall instances configured as back-end pool members

#### Routing table configuration
{: #rmnlb-routing-table-configuration}

Egress routing table (client VPC):

* Destination: Server subnet CIDR
* Next hop: NLB IP address
* Action: Deliver

Egress routing table (server VPC):

* Destination: Client subnet CIDR
* Next hop: NLB IP address
* Action: Deliver

#### Performance characteristics
{: #rmnlb-performance-characteristics}

Consider the following performance characteristics:

* Lower latency than application load balancers
* Horizontal scaling by adding firewall instances
* Session persistence helps ensure that stateful inspection works correctly
* Throughput scales with the number of firewall instances
* Works with any firewall vendor

#### Benefits
{: #rmnlb-benefits}

Route mode provides the following benefits:

Scalability
:   Add firewall instances to increase capacity.

High availability
:   Multiple active instances provide redundancy.

Flexibility
:   Works with any firewall vendor.

Simplicity
:   Requires no complex BGP configuration.

Performance
:   Efficient Layer 4 load balancing.

For more information, see [Virtual firewall appliances with network load balancer for traffic management](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc#Virtual-firewall-Appliances-with-NLB).

### Custom images, BYOA, and vendor support
{: #custom-images-byoa-vendor-support}

In addition to IBM-provided catalog images, {{site.data.keyword.vpc_short}} supports customer-managed custom images for firewall and network virtual appliance deployments. Custom images enable BYOA deployments, so you can use other vendors and appliance configurations beyond the currently published marketplace offerings. For more information, see [Getting started with custom images](/docs/vpc?topic=vpc-planning-custom-images).

PayGo offerings, such as the [Fortinet FortiGate licensed firewall](/docs/licensed-firewall), include the license and support and are the recommended starting point for new Fortinet deployments. BYOL offerings in the {{site.data.keyword.cloud_notm}} catalog are vendor-supported images where you supply the license but deploy from an approved catalog tile. BYOA deployments use custom images outside the catalog and require extra validation. They do not imply vendor certification or {{site.data.keyword.cloud_notm}} support. You are responsible for obtaining and maintaining any required virtual appliance images, licenses, and support entitlements directly from the appliance vendor by using your existing vendor account or procurement channel. Support for custom images depends on the individual vendor's support policies for {{site.data.keyword.cloud_notm}} deployments, and not all vendors officially certify or support their virtual appliances on {{site.data.keyword.cloud_notm}} infrastructure.
{: note}

Common use cases include:

* Deploying vendor-supported BYOL virtual appliances that are not yet available in the {{site.data.keyword.cloud_notm}} catalog
* Using customized firewall builds with preinstalled policies or automation tools
* Supporting more network security vendors and virtual routers
* Migrating existing appliance images from on-premises or other cloud environments

You are also responsible for validating vendor compatibility, licensing, and automation behavior when you use custom images. Features such as SDN Connector integration, automated failover, or public address range automation might require vendor-specific implementation or customer-managed automation.

Supported HA deployment patterns vary by vendor. Before you implement a BYOA deployment, review the [vendor support matrix for HA firewall deployments](#vendor-support-matrix-firewall-deployment-licensing).

## Licensing models
{: #licensing-models}

### Available licensing models
{: #licensing-current}

Two licensing models are available for {{site.data.keyword.vpc_short}} firewalls:

PayGo (recommended)
:   The license and FortiCare Premium support are included with the offering and billed through {{site.data.keyword.cloud_notm}}. IBM manages the full license lifecycle, including provisioning, validation, and renewal. No separate vendor procurement or tracking is required. Three Fortinet FortiGate VM PayGo offerings are available in the {{site.data.keyword.cloud_notm}} catalog, covering all supported deployment topologies. For more information, see [Fortinet FortiGate VM PayGo (IBM-licensed firewall offering)](#paygo).

BYOL
:   You are responsible for firewall licensing, software maintenance, signature updates, and vendor lifecycle management unless otherwise specified by the vendor offering.

## Migration from {{site.data.keyword.cloud_notm}} Classic
{: #migration-from-classic}

### Classic infrastructure overview
{: #classic-infrastructure}

{{site.data.keyword.cloud_notm}} Classic infrastructure uses a Layer 2 network architecture with gateway-based firewall appliances to provide routing, security, and traffic inspection between VLANs and external networks.

Classic environments commonly rely on gateway appliances for centralized traffic control and security enforcement.

#### Gateway appliances
{: #gateway-appliances}

The following gateway appliance offerings are available in {{site.data.keyword.cloud_notm}} Classic infrastructure:

* Virtual FortiGate (vFSA) (see [Getting started with Fortigate Security Appliance](/docs/vfsa?topic=vfsa-getting-started-vfsa))
* Virtual Juniper vSRX (see [Getting started with {{site.data.keyword.cloud_notm}} Juniper vSRX](/docs/vsrx?topic=vsrx-getting-started-vsrx))
* Virtual Router Appliance (VRA (Vyatta)) (see [Getting started with IBM Virtual Router Appliance](/docs/virtual-router-appliance?topic=virtual-router-appliance-getting-started-vra))

For more information, see [Getting started with {{site.data.keyword.cloud_notm}} Gateway Appliance](/docs/gateway-appliance?topic=gateway-appliance-getting-started-ga).
{: note}

#### Deprecated physical firewalls
{: #deprecated-firewalls}

The following physical firewall offerings are deprecated:

* FortiGate 10G (see [Exploring firewalls](/docs/fortigate-10g?topic=fortigate-10g-exploring-firewalls))
* Hardware Firewall (Shared) (see [Getting started with Hardware Firewall](/docs/hardware-firewall-shared?topic=hardware-firewall-shared-deprecation-hardware-shared-firewall))

### Classic to VPC firewall and networking mapping
{: #classic-to-vpc-mapping}

The following table maps {{site.data.keyword.cloud_notm}} Classic gateway-based firewall models to equivalent {{site.data.keyword.vpc_short}} networking and firewall constructs.

The architectural differences between Layer 2 (Classic) and Layer 3 SDN (VPC) environments are reflected in the table. In VPC, routing, segmentation, and security are separated into distinct services rather than centralized in a single gateway appliance.

| Classic offering | Primary role in Classic | VPC equivalent construct | VPC service / offering |
| ----------------- | ------------------------ | -------------------------- | ------------------------ |
| Vyatta / Virtual Router Appliance (VRA) | Central routing, NAT, basic firewalling, inter-VLAN connectivity | Distributed routing, segmentation, and security controls | Security Groups, Network ACLs, Public Gateway, Transit Gateway |
| Virtual Juniper vSRX | Advanced firewall with routing and inspection | Dedicated virtual firewall appliance | Juniper vSRX BYOL deployment on VPC virtual server instances or bare metal servers |
| Virtual FortiGate (vFSA) | NGFW, IPS/IDS, VPN termination, centralized inspection | Dedicated NGFW appliance | Fortinet FortiGate BYOL or PayGo deployment on VPC virtual server instances or bare metal servers |
| Gateway Appliance (general concept) | Combined routing, security, and connectivity hub | Decoupled routing and security architecture | Transit Gateway, VPN Gateway, and optional firewall inspection VPC |
| Hardware Firewall (Shared) | Managed perimeter firewall | Native VPC security controls or dedicated firewall appliances | Security Groups, Network ACLs, {{site.data.keyword.cis_short}}, or VPC firewall appliances |
{: caption="Classic to VPC firewall and networking mapping" caption-side="bottom"}

### Key differences between Classic and VPC
{: #classic-vs-vpc}

The following table highlights the key architectural and operational differences between {{site.data.keyword.cloud_notm}} Classic infrastructure and {{site.data.keyword.vpc_short}}.

| Aspect | {{site.data.keyword.cloud_notm}} Classic | {{site.data.keyword.vpc_short}} |
| -------- | ------------------- | --------------- |
| Network architecture | Layer 2 | Layer 3 SDN |
| Licensing | IBM-licensed (except BYOL Gateway) | BYOL or PayGo (IBM-licensed PayGo now available for Fortinet FortiGate) |
| High availability | Active/Passive Single Data Center Pod | Multiple HA patterns |
| Deployment flexibility | Physical and bare metal | Virtual (virtual server instance and bare metal server) |
{: caption="Key differences between Classic and VPC infrastructure" caption-side="bottom"}

### Migration considerations for gateway devices
{: #migration-considerations}

Consider the following factors when you migrate gateway-based firewall deployments from Classic infrastructure to VPC:

1. Review the VPC Layer 3 SDN architecture requirements and adapt your network design patterns accordingly.
1. If you are migrating from Classic VLAN-based routing and inspection deployments, evaluate VPC custom routing tables, Transit Gateway, and Transit VPC architectures to replace these patterns.
1. Plan for BYOL requirements or IBM-licensed options.
1. Choose the appropriate HA pattern based on your requirements.
1. Use Terraform&reg; and APIs for deployment.
1. Thoroughly test failover scenarios in the VPC environment.

## Choosing the right deployment option
{: #choosing-deployment}

Performance, scalability, operational requirements, and cost all affect which firewall deployment option is appropriate for your workload. Review the key factors that can affect firewall performance before you select a deployment configuration.

### Performance factors
{: #performance-factors}

Multiple factors influence firewall performance in {{site.data.keyword.vpc_short}}. Review these factors before you select a deployment configuration.

#### Firewall license
{: #firewall-license-performance}

Firewall vendor licenses typically determine the maximum throughput that a firewall can achieve.

Consider the following licensing factors:

vCPU count
:   Firewall vendor licenses typically limit throughput based on the number of licensed vCPUs.

Gating factor
:   The license vCPU limit is usually the primary constraint on performance.

Example
:   A 4-vCPU firewall license limits throughput regardless of virtual server instance profile size.

Recommendation
:   Match virtual server instance profile vCPU count to firewall license vCPU entitlement.

#### Virtual server instance profile selection
{: #vsi-profile-selection}

The selected virtual server instance profile affects the available network bandwidth and overall firewall performance.

Consider the following profile characteristics:

Profile size
:   Larger profiles provide higher bandwidth limits. For more information, see [x86-64 instance profiles](/docs/vpc?topic=vpc-profiles) and [Gen 4 profile examples](#gen4-vsi-profiles).

Bandwidth pooling (Gen 4 profiles only)
:   Network bandwidth is pooled across all interfaces, which allows flexible allocation.

Pre-Gen 4 profiles
:   Available bandwidth is divided equally across interfaces and is not pooled.

Example
:   A `bx4-32x128` profile provides a 64 Gbps bandwidth limit that can be pooled across all interfaces (Gen 4).

PayGo profile assignments
:   If you deploy an IBM-licensed PayGo offering, the virtual server instance profile (Gen 2 or Gen 3 compute-optimized `cx` profiles) is automatically assigned based on your selected license plan and deployment size, and cannot be changed or upgraded to a Gen 4 profile. All license plans are available in all supported regions. In a few regions, the assigned VPC profile might differ from the default, but the deployment size and entitlement remain equivalent. For details on these automatic mappings, see [About firewall license plans and instance profiles](/docs/licensed-firewall?topic=licensed-firewall-about-firewall-license-plans-and-instance-profiles).

#### Network interface configuration
{: #network-interface-config}

The number and configuration of network interfaces can affect available bandwidth.

Consider the following factors:
Number of interfaces
:   More interfaces can affect available bandwidth per interface on pre-Gen 4 profiles.

Gen 4 advantage
:   Bandwidth pooling eliminates per-interface division.

Recommendation
:   Use Gen 4 profiles for firewall deployments when they are available.

#### Network load balancer (Active/Active)
{: #nlb-active-active}

Active/Active deployments that use an NLB introduce additional performance considerations.

Consider the following factors:

NLB throughput
:   An NLB has its own throughput limits.

Route mode
:   Provides lower latency than application load balancers and enables efficient Layer 4 routing.

Scaling
:   You can distribute traffic across multiple firewall instances to increase aggregate throughput.

In some virtual firewall implementations, GRE encapsulation and processing might reach CPU capacity limits, which restricts throughput even when network capacity is sufficient.

#### Bare metal servers
{: #bare-metal-performance}

Bare metal servers provide dedicated hardware resources and can deliver consistent network performance.

Consider the following characteristics:
High network throughput
:   Dedicated hardware resources provide consistent high performance.

Virtual network floating interfaces
:   Automatic failover without SDN Connector overhead.

Significant limitations
:   Requires manual hypervisor and VM configuration, monthly billing only, limited scaling flexibility, and customer-managed OS and software.

Recommendation
:   Consider virtual server instance deployments first because of their ease of operation and flexibility.

#### Key recommendations
{: #performance-recommendations}

The following recommendations can help optimize firewall deployments in VPC:

1. Ensure that the virtual server instance profile vCPU count matches or exceeds the firewall license vCPU entitlement.
1. Use Gen 4 profiles (applicable to BYOL or BYOA deployments) because bandwidth pooling provides better flexibility for multi-interface firewalls. IBM-licensed PayGo offerings are automatically mapped to Gen 2 or Gen 3 `cx` profiles and do not support Gen 4.
1. Start with virtual server instance deployments because they offer better flexibility, easier management, and hourly billing.
1. Validate that actual throughput meets requirements before production deployment.

#### Gen 4 virtual server instance profile examples
{: #gen4-vsi-profiles}

Balanced profiles (`bx4`)
:   The following examples show Gen 4 balanced profiles:

| Profile | vCPU | Memory (GiB) | Bandwidth cap (Gbps) |
| --------- | ------ | -------------- | ---------------------- |
| `bx4-8x32` | 8 | 32 | 16 |
| `bx4-16x64` | 16 | 64 | 32 |
| `bx4-32x128` | 32 | 128 | 64 |
| `bx4-48x192` | 48 | 192 | 96 |
| `bx4-64x256` | 64 | 256 | 128 |
{: caption="Gen 4 balanced virtual server instance profiles (bx4)" caption-side="bottom"}

Gen 4 profiles feature bandwidth pooling across all interfaces. For complete profile details, see [General-purpose instance profiles - Intel Gen 4](/docs/vpc?topic=vpc-general-purpose-vsi-profiles-gen4-intel).
{: note}

### Cost considerations
{: #cost-considerations}

Cost varies depending on the selected deployment pattern and infrastructure model.

Consider the following cost factors:
Stand-alone
:   Minimal cost and no redundancy.

Single-zone HA
:   Moderate cost with a balance between resilience and complexity.

Multizone HA
:   Higher cost with greater availability.

Virtual server instance versus bare metal server
:   Virtual server instances offer hourly billing and operational flexibility. Bare metal servers require monthly billing and manual management.

PayGo versus BYOL
:   PayGo includes the firewall license and FortiCare Premium support in the hourly billing rate. No upfront license purchase or renewal management is required. BYOL requires a separate license purchase from the vendor, which adds procurement overhead but might suit you if you have existing vendor license agreements.

## Other resources
{: #other-resources}

### IBM TechXchange blogs
{: #techxchange-blogs}

[IBM TechXchange firewall blog posts](https://community.ibm.com/community/user/viewdocument/firewall-deployment-library-for-ibm-cloud-vpc?CommunityKey=dd1ee2bc-c83b-4afb-bd1c-9095ff0c3bc1&tab=librarydocuments){: external} provide technical walkthroughs and deployment guidance for firewall solutions in {{site.data.keyword.vpc_short}}.

### {{site.data.keyword.cloud_notm}} documentation

- [Fortinet vFSA on IBM Cloud: Bare-Metal HA](https://community.ibm.com/community/user/blogs/andrew-sloma/2026/07/02/fortinet-vfsa-on-ibm-cloud-bm-ap){: external}

- [Fortinet vFSA on IBM Cloud: OnPrem to Spoke VPC with Active/Active/Active HA](https://community.ibm.com/community/user/blogs/andrew-sloma/2026/08/04/fortinet-vfsa-on-ibm-cloud-onprem-to-vpc-aaa){: external}

- [Fortinet vFSA on IBM Cloud: Single VPC Active/Active with Route Mode NLB](https://community.ibm.com/community/user/blogs/andrew-sloma/2026/08/28/fortinet-vfsa-on-ibm-cloud-single-vpc-aa){: external}

- [Fortinet vFSA on IBM Cloud: Single Zone Active/Passive for Hub-And-Spoke](https://community.ibm.com/community/user/blogs/andrew-sloma/2026/09/02/fortinet-vfsa-on-ibm-cloud-singlezone-ap){: external}

- [Fortinet vFSA on IBM Cloud: Cross Zone Active/Passive for Hub-And-Spoke](https://community.ibm.com/community/user/blogs/andrew-sloma/2026/10/02/fortinet-vfsa-on-ibm-cloud-crosszone-ap){: external}

### IBM Cloud documentation
{: #cloud-docs}

The following {{site.data.keyword.cloud_notm}} documentation topics provide additional information about firewall deployment, VPC networking, and related services.

* [Fortinet FortiGate licensed firewall for IBM Cloud VPC](/docs/licensed-firewall?topic=licensed-firewall-getting-started)
* [Transit VPC pattern](/docs/pattern-transit-vpc?topic=pattern-transit-vpc-transit-vpc)
* [VPC networking overview](/docs/vpc?topic=vpc-about-networking-for-vpc)

### Support
{: #ibm-support}

{{site.data.keyword.cloud_notm}} support
:   For infrastructure and platform issues. For PayGo offerings, IBM also provides support for licensing lifecycle issues. To open a case, see [Getting help and support](/docs/licensed-firewall?topic=licensed-firewall-help-and-support).

Vendor support
:   For firewall configuration and BYOL licensing.

## Summary
{: #summary}

{{site.data.keyword.vpc_short}} offers flexible firewall deployment options to meet diverse security and availability requirements. Whether you need a stand-alone deployment for development or a complex multizone Active/Active configuration for enterprise workloads, VPC provides the tools and patterns required.

For most new deployments, the Fortinet FortiGate PayGo offering is the recommended choice. It includes the license and FortiCare Premium support, billed through {{site.data.keyword.cloud_notm}}, and is available in three topologies:

Single VM
:   A single FortiGate instance with no redundancy. Suitable for development, testing, or noncritical workloads.

Active/Passive HA (single zone)
:   Two FortiGate instances in the same availability zone as an active-passive cluster. The IBM Cloud SDN Connector provides automatic failover if the active node fails.

Active/Passive HA (Cross Zone)
:   Extends the single-zone HA topology across two availability zones. A public address range provides a VPC-scoped public IP that can be rebound across zones, which provides resilience against a full zone outage. This is the highest-availability configuration.

To get started, see [Getting started with Fortinet FortiGate for IBM Cloud VPC](/docs/licensed-firewall?topic=licensed-firewall-getting-started).

For assistance with firewall deployment planning or migration from Classic infrastructure, contact [{{site.data.keyword.cloud_notm}} support](/docs/licensed-firewall?topic=licensed-firewall-help-and-support) or your IBM representative.
