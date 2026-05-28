---

copyright:
  years: 2026
lastupdated: "2026-05-28"

keywords: direct link migration, direct link on classic, direct link 1.0, direct link 2.0, BGP migration, traffic cutover, network migration, dl migration, dl 1.0 to 2.0

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from Direct Link on Classic (1.0) to Direct Link (2.0)
{: #migrate-directlink-1-to-2}

Learn how to migrate traffic from {{site.data.keyword.dlc_short}} (1.0) to {{site.data.keyword.dl_short}} (2.0) on {{site.data.keyword.cloud}} by using either a graceful traffic shift or a hard cutover.
{: shortdesc}

{{site.data.keyword.dlc_short}} refers to {{site.data.keyword.dl_short}} version 1.0, while {{site.data.keyword.dl_short}} refers to version 2.0, the current, actively offered service.
{: note}

## Why migrate to Direct Link (2.0)?
{: #why-migrate}

{{site.data.keyword.dl_short}} offers the following improvements over {{site.data.keyword.dlc_short}}:

No-cost global routing
:   Includes bring your own IP (BYOIP) support.

Flexible bandwidth options
:   {{site.data.keyword.dl_short}} Connect supports 50 Mbps to 5 Gbps. {{site.data.keyword.dl_short}} Dedicated supports 1 Gbps to 10 Gbps.

Integrated support
:   {{site.data.keyword.dl_short}} provides for virtual routing, dynamic path selection, and Virtual Private Cloud (VPC) connectivity.

Metered billing
:   No price increase compared to {{site.data.keyword.dlc_short}}.

Automated ordering experience
:   API integration with partners streamlines the ordering process.

Additional features
:   {{site.data.keyword.dl_short}} includes additional capabilities not available in {{site.data.keyword.dlc_short}}, such as MACsec encryption support in select locations, enhanced monitoring, and improved API functions.

These enhancements make {{site.data.keyword.dl_short}} the recommended solution for connecting your on-premises infrastructure to {{site.data.keyword.cloud_notm}}.

For a detailed comparison of all features and capabilities between {{site.data.keyword.dlc_short}} and {{site.data.keyword.dl_short}}, including location-specific features like MACsec, see [Feature comparison between {{site.data.keyword.dl_short}} versions](/docs/dl?topic=dl-dl-comparison-classic).
{: tip}

## Before you begin
{: #before-you-begin}

Review the following prerequisites before you start your {{site.data.keyword.dl_short}} migration.

{{site.data.keyword.dlc_short}} and {{site.data.keyword.dl_short}} are managed in different areas of the {{site.data.keyword.cloud_notm}} console. During migration, you must access both areas to manage your existing 1.0 connection and provision your new 2.0 connection.
{: note}

### Verify your Direct link configuration
{: #verify-dl-config}

Before you proceed with migration, verify that your current {{site.data.keyword.dlc_short}} connection uses Border Gateway Protocol (BGP) routing.

{{site.data.keyword.dl_short}} does not support static routing. If your {{site.data.keyword.dlc_short}} connection uses static routes instead of BGP, you cannot use the procedures in this document. {{site.data.keyword.dl_short}} requires BGP for all connections.
{: important}

### Required access
{: #required-access}

Make sure that you have the following access:

- Administrative access to your on-premises Customer Edge (CE) router
- {{site.data.keyword.cloud_notm}} account access with the appropriate {{site.data.keyword.iamshort}} permissions for {{site.data.keyword.dl_short}}
- Coordination established with your telco/Network Service Provider (NSP) partner

### Required skills
{: #required-skills}

The following skills are required:

- Experience configuring and troubleshooting BGP
- Understanding of routing protocols and traffic engineering
- Familiarity with your router vendor's Command Line Interface (CLI) (for example, Cisco, Juniper, or Arista)
- Knowledge of your {{site.data.keyword.dlc_short}} configuration
- Understanding of {{site.data.keyword.dl_short}} features

The procedures in this document outline general configuration and verification steps. You must adapt these steps to your specific router vendor's command syntax and configuration methods. Consult your router vendor's documentation for the exact commands and procedures.
{: important}

### Planning requirements
{: #planning-requirements}

Complete the following planning tasks:

- If your {{site.data.keyword.dl_short}} is connected to the Classic network, make sure that your account is Virtual Routing and Forwarding (VRF)-enabled. For more information, see [Enabling VRF and service endpoints](/docs/account?topic=account-vrf-service-endpoint) and the [VRF FAQs](/docs/account?topic=account-vrf-faqs).
- Plan a maintenance window (for hard cutover scenario).
- Document rollback procedures.
- Set up network monitoring tools.
- Create a stakeholder communication plan.

## Choosing your migration scenario
{: #choosing-scenario}

Choose the appropriate migration scenario based on your network architecture and business requirements. Use the following decision criteria.

Use graceful migration when:

- You have separate physical interfaces on your on-premises customer edge (CE) router, or separate on-premises CE routers, for each {{site.data.keyword.dl_short}} connection.
- You require zero or minimal downtime.
- You want to validate the new link with production traffic before full cutover.
- You have time to support a phased migration approach.

Use hard cutover when:

- Both links must share the same physical path back to your on-premises customer equipment.
- Both links must use the same IP addresses.
- Your on-premises customer edge (CE) router does not have enough available ports to connect both the new and old {{site.data.keyword.dl_short}} connections simultaneously.
- Network limitations prevent running parallel paths.
- You have a suitable maintenance window available.
- You prefer a simpler, faster cutover process.

If you have backup connectivity, such as a VPN connection to {{site.data.keyword.cloud_notm}}, you can use it as a fallback during the cutover window to minimize the impact of any outage.
{: tip}

## Migration scenarios
{: #migration-scenarios}

Scenario 1: Graceful traffic shift
:   Recommended for most deployments. This scenario allows for controlled migration with minimal disruption by running both links in parallel during the transition.

Scenario 2: Hard cutover
:   Required when both links must share the same physical path, use the same IP addresses, or when your router lacks available ports for simultaneous connections. This scenario involves a planned outage window.

## Scenario 1: Graceful traffic shift migration
{: #graceful-migration}

In this scenario, you can shift traffic gradually from {{site.data.keyword.dlc_short}} to {{site.data.keyword.dl_short}} without a hard cutover, minimizing disruption to your services.

The following diagram illustrates the graceful migration process, where both connections remain active while traffic is gradually shifted from {{site.data.keyword.dlc_short}} to {{site.data.keyword.dl_short}}.

![Graceful traffic shift migration](/images/dl_migration-scenario_1.svg){: caption="Graceful traffic shift migration" caption-side="bottom"}

### Phase 1: Preparation and provisioning
{: #graceful-phase-1}

In this phase, prepare your environment for migration by provisioning {{site.data.keyword.dl_short}} resources and configuring the required network connectivity, routing, and gateway settings for a smooth transition.

#### Order and provision Direct Link
{: #order-provision-dl2}

Order and provision a new {{site.data.keyword.dl_short}} instance in the {{site.data.keyword.cloud_notm}} console. Make sure it meets your network connectivity and performance requirements.

1. In the {{site.data.keyword.cloud_notm}} console, order a new {{site.data.keyword.dl_short}} instance that matches your requirements.
   - For {{site.data.keyword.dl_short}} Dedicated, see [Ordering {{site.data.keyword.dl_short}} Dedicated](/docs/dl?group=ordering-ibm-cloud-direct-link-dedicated).
   - For {{site.data.keyword.dl_short}} Connect, see [Ordering {{site.data.keyword.dl_short}} Connect](/docs/dl?group=ordering-ibm-cloud-direct-link-connect).

      For Connect orders, you might need to complete provisioning through your provider's portal.
      {: note}

1. Specify the following details:
   - **Location (PoP)**
   - **Speed**
   - **Telco/NSP Partner**
   - **Routing option** (Local is default, Global if needed)
1. Provide {{site.data.keyword.cloud_notm}} with necessary connection details (for example, a Letter of Authorization (LOA) or Connecting Facility Assignment (CFA) if Dedicated).
1. Work with your telco/NSP partner to provision their side of the circuit or connection to the specified {{site.data.keyword.cloud_notm}} PoP Meet-Me Room (MMR) or location.

#### Configure physical and logical connectivity
{: #configure-connectivity}

Establish the underlying network connectivity by configuring your on-premises router and verifying that the physical and Layer 2 or Layer 3 connections to {{site.data.keyword.cloud_notm}} are properly set up.

1. Configure the physical interface on your on-premises router:
   - Set an IP address.
   - Configure Maximum Transmission Unit (MTU) (typically 1500 or higher if your network supports it end-to-end). Consider the use of jumbo frames (MTU 9000) for improved performance if your network path and {{site.data.keyword.cloud_notm}} configuration support them.
1. Confirm that Layer 2 or Layer 3 connectivity is established between your router interface and the NSP demarcation point leading to {{site.data.keyword.cloud_notm}}.

#### Configure the Direct Link gateway
{: #configure-dl2-gateway}

After provisioning connectivity, configure your {{site.data.keyword.dl_short}} gateway in the {{site.data.keyword.cloud_notm}} console to define routing, connection types, and integration with your network architecture.

In the {{site.data.keyword.cloud_notm}} console, configure your {{site.data.keyword.dl_short}} gateway with the following details:

- Your BGP ASN (Customer ASN)
- BGP peering IP addresses (usually a `/30` or `/31` subnet that is provided by IBM)
- Optional: BGP MD5 Authentication key
- Connection type - choose one of the following options:
   - Virtual Connections to VPCs or Classic Infrastructure
   - {{site.data.keyword.tg_short}} connection for centralized connectivity to multiple network types

When you connect to a {{site.data.keyword.tg_short}}, the {{site.data.keyword.dl_short}} communicates exclusively with the {{site.data.keyword.tg_short}}, which then provides connectivity to all attached networks, including VPCs, Classic Infrastructure, Generic Routing Encapsulation (GRE) tunnels, and {{site.data.keyword.powerSys_notm}}. This architecture simplifies network management and provides a hub-and-spoke topology.
{: tip}

#### Configure on-premises router BGP peering
{: #configure-onprem-bgp}

Set up BGP peering on your on-premises router to enable route exchange with {{site.data.keyword.cloud_notm}} and prepare for controlled shifting of the traffic during the migration.

1. Configure the interface IP address that matches the peering subnet that is provided by IBM.
1. Define the BGP neighbor relationship toward the {{site.data.keyword.cloud_notm}} BGP peer IP addresses for the {{site.data.keyword.dl_short}} link.
1. Configure your BGP ASN and apply the MD5 key if used.
1. Prepare inbound and outbound BGP route maps/policies specifically for this new {{site.data.keyword.dl_short}} neighbor. These policies are used for traffic engineering during the migration.

{{site.data.keyword.dl_short}} provides built-in capabilities for route filtering and AS path prepending, simplifying your BGP configuration. For more information, see [Setting up route filters](/docs/dl?group=setting-up-route-filters) and [Prepending AS paths](/docs/dl?topic=dl-prepend-as-paths).
{: tip}

### Phase 2: Establishing parallel path and verification
{: #graceful-phase-2}

In this phase, you bring up the new {{site.data.keyword.dl_short}} connection alongside your old {{site.data.keyword.dlc_short}} and validate that it is functioning correctly before you make a shift in any production traffic.

#### Configure initial BGP policy (make Direct Link less preferred)
{: #initial-bgp-policy}

Configure BGP policies to help ensure {{site.data.keyword.dlc_short}} remains the active path while {{site.data.keyword.dl_short}} is being verified.

**Outbound policy (On-Premises → IBM):**

Choose one of the following options:

* **Option A (recommended)**: Advertise your on-premises prefixes with significant `AS_PATH` prepending (for example, prepend your own ASN three to five times). This option makes the path longer and less desirable for {{site.data.keyword.cloud_notm}} return traffic compared to the {{site.data.keyword.dlc_short}} path.
* **Option B (more control)**: Initially, do not advertise any production prefixes over {{site.data.keyword.dl_short}}. Advertise only a specific test prefix if needed.

{{site.data.keyword.dl_short}} supports AS path prepending directly through the console or API. For details, see [Prepending AS paths](/docs/dl?topic=dl-prepend-as-paths).
{: tip}

**Inbound policy (IBM → On-Premises):**

1. Accept the {{site.data.keyword.cloud_notm}} prefixes (VPC prefixes, classic subnets, {{site.data.keyword.tg_short}} routes).
1. Set the `local-preference` for these received routes to a value lesser than the `local-preference` assigned to routes received from the {{site.data.keyword.dlc_short}} peer. For example,

   * {{site.data.keyword.dlc_short}} = `100`
   * {{site.data.keyword.dl_short}} = `90`
1. This configuration makes sure your outbound traffic from on-premises still prefers the {{site.data.keyword.dlc_short}} link.

You can also use {{site.data.keyword.dl_short}}'s route filtering capabilities to control which routes are learned. For more information, see [Setting up route filters](/docs/dl?group=setting-up-route-filters).
{: tip}

If you use {{site.data.keyword.tg_short}}, you receive aggregated routes for all networks that are attached to the {{site.data.keyword.tg_short}}, simplifying your routing table.
{: note}

#### Establish and verify BGP session
{: #establish-bgp-session}

Enable and confirm the BGP session between your on-premises router and {{site.data.keyword.cloud_notm}} to make sure that the new connection is stable and operational.

1. Enable the BGP neighbor configuration on your on-premises router for {{site.data.keyword.dl_short}}.
1. Verify that the BGP session comes up by checking the BGP summary and neighbor status on your router.
1. Verify that the BGP session state is `Established`.
1. Check the router logs for any BGP errors.

#### Verify route exchange and policies
{: #verify-route-exchange}

After the BGP session is established, verify that routes are being correctly advertised and received, and that your routing policies are working as intended.

1. Check routes that are received from {{site.data.keyword.cloud_notm}} through the {{site.data.keyword.dl_short}} BGP neighbor. Confirm that you see the expected {{site.data.keyword.cloud_notm}} prefixes and that your inbound policy (low `local-preference`) is applied.

1. Check routes that are advertised toward {{site.data.keyword.cloud_notm}} through the {{site.data.keyword.dl_short}} BGP neighbor. Confirm your outbound policy (`AS_PATH` prepending or limited prefixes) is applied.

1. Check your overall BGP table. Confirm that {{site.data.keyword.cloud_notm}} prefixes are learned through both {{site.data.keyword.dlc_short}} and {{site.data.keyword.dl_short}}, and that the {{site.data.keyword.dlc_short}} path is marked as the `best` path due to the higher `local-preference`.

1. Check your routing table to make sure routes toward {{site.data.keyword.cloud_notm}} still point to the {{site.data.keyword.dlc_short}} next-hop.

#### Run connectivity tests
{: #connectivity-tests}

Test connectivity and performance over the new {{site.data.keyword.dl_short}} path to confirm it is ready to handle traffic when you begin the migration.

1. Run basic `pings` and `traceroutes` that are sourced from the {{site.data.keyword.dl_short}} interface IP on your router toward the {{site.data.keyword.cloud_notm}} BGP peer IP to validate Layer 3 reachability over the new path.
1. If possible, `ping` a test VM inside the connected {{site.data.keyword.cloud_notm}} VPC by using the {{site.data.keyword.dl_short}} path (might require temporarily advertising a specific test host route over {{site.data.keyword.dl_short}} only, or by using advanced VRF or policy-based routing features on your router).
1. Consider running performance tests by using your preferred tools such as `iperf`, `netperf`, or other bandwidth and latency testing tools to validate throughput and latency characteristics of the new {{site.data.keyword.dl_short}} path before migrating production traffic.

At this stage, {{site.data.keyword.dl_short}} is up and peered, but {{site.data.keyword.dlc_short}} is still carrying all production traffic.
{: note}

### Phase 3: Traffic migration (maintenance window recommended)
{: #graceful-phase-3}

In this phase, you gradually shift production traffic from {{site.data.keyword.dlc_short}} to {{site.data.keyword.dl_short}} by adjusting routing policies, ideally during a maintenance window to minimize impact.

#### Shift outbound traffic (On-Premises → IBM)
{: #shift-outbound-traffic}

Update your routing policies so that outbound traffic from your on-premises environment prefers the {{site.data.keyword.dl_short}} path.

1. Modify the inbound BGP policy on your on-premises router for routes received from {{site.data.keyword.dl_short}}.
1. Increase the `local-preference` for the routes that are received by using {{site.data.keyword.dl_short}} to be higher than the `local-preference` for routes that are received by using {{site.data.keyword.dlc_short}}. For example,

   * {{site.data.keyword.dl_short}} = `110`
   * {{site.data.keyword.dlc_short}} = `100`
1. Apply the policy change by running a soft reset of the BGP session for the {{site.data.keyword.dl_short}} neighbor (inbound direction).

1. Verify your BGP and routing tables. Routes toward {{site.data.keyword.cloud_notm}} now show the {{site.data.keyword.dl_short}} next-hop as the best path.

1. Monitor interface counters, NetFlow, and other metrics to confirm the traffic is egressing your router toward {{site.data.keyword.cloud_notm}} starts flowing over the {{site.data.keyword.dl_short}} interface.

#### Shift inbound traffic (IBM → On-Premises)
{: #shift-inbound-traffic}

Adjust your route advertisements to influence return traffic from {{site.data.keyword.cloud_notm}}, make sure it shifts to the {{site.data.keyword.dl_short}} path.

1. Modify the outbound BGP policy on your on-premises router for prefixes advertised toward {{site.data.keyword.cloud_notm}}.
1. Reduce or remove the `AS_PATH` prepending on prefixes that are advertised over {{site.data.keyword.dl_short}}.
1. Optionally (for more certainty): Increase the `AS_PATH` prepending for prefixes that are advertised over {{site.data.keyword.dlc_short}} simultaneously, or just before you remove it from {{site.data.keyword.dl_short}} advertisements.
1. Apply the policy change by performing a soft reset of the BGP session for the {{site.data.keyword.dl_short}} neighbor (outbound direction).
1. Monitor inbound traffic on your on-premises router. Traffic arriving over the {{site.data.keyword.dl_short}} interface increases as traffic over {{site.data.keyword.dlc_short}} decreases. The shift can take a short time due to BGP propagation within {{site.data.keyword.cloud_notm}}.

Consider the use of {{site.data.keyword.dl_short}} 2.0's native AS path prepending feature for ongoing traffic engineering. For more information, see [Prepending AS paths](/docs/dl?topic=dl-prepend-as-paths).
{: tip}

#### Verify and monitor
{: #verify-monitor-graceful}

Monitor traffic patterns and application performance to confirm that traffic is migrated successfully and that the new path is stable.

1. Continuously monitor interface traffic rates on both {{site.data.keyword.dlc_short}} and {{site.data.keyword.dl_short}} interfaces on your router.
1. Check application performance and connectivity for services that traverse the {{site.data.keyword.dl_short}}.
1. Use traceroutes from both on-premises and {{site.data.keyword.cloud_notm}} instances to confirm that the new path is being used.

Traffic now flows primarily over the {{site.data.keyword.dl_short}} link.
{: note}

### Phase 4: Decommissioning Direct Link on Classic
{: #graceful-phase-4}

In this final phase, retire the {{site.data.keyword.dlc_short}} connection after you confirm that all traffic is successfully running over {{site.data.keyword.dl_short}}.

#### Stabilization period
{: #stabilization-period}

Run traffic over {{site.data.keyword.dl_short}} for a defined period, for example, 24 to 48 hours or according to your policy, to ensure stability and to handle unforeseen issues.

#### Gracefully shut down Direct Link on Classic BGP
{: #shutdown-dl1-bgp}

Shut down the BGP session for the {{site.data.keyword.dlc_short}} connection to stop route exchange.

1. On your on-premises router, administratively shut down the BGP neighbor session toward the {{site.data.keyword.dlc_short}} peer.

1. Verify that the BGP session goes down.

#### Remove Direct Link on Classic configuration
{: #remove-dl1-config}

After you confirm stability, remove the BGP neighbor configuration and related route maps or policies for the {{site.data.keyword.dlc_short}} connection from your on-premises router. Optionally keeping the interface configuration as a temporary fallback.

#### Deprovision Direct Link on Classic
{: #deprovision-dl1}

Delete the {{site.data.keyword.dlc_short}} resources in {{site.data.keyword.cloud_notm}} and coordinate with your provider to disconnect the physical circuit.

1. In the {{site.data.keyword.cloud_notm}} console, delete the {{site.data.keyword.dlc_short}} gateway instance. Deleting the gateway detaches it from VPCs or Classic infrastructure.
1. Inform your telco/NSP partner to formally decommission the {{site.data.keyword.dlc_short}} circuit.

#### Final cleanup
{: #final-cleanup-graceful}

Complete the process by removing any remaining interface configuration related to {{site.data.keyword.dlc_short}} on your on-premises router. Also, update network diagrams and documentation.

## Scenario 2: Hard cutover migration
{: #hard-cutover-migration}

This scenario is required when the new and old links must share the same physical path or IP addresses, or when your router lacks enough ports to support both {{site.data.keyword.dl_short}} connections simultaneously. This method involves a planned outage window where the old link is deactivated and the new link is activated for production traffic almost simultaneously.

The following diagram illustrates the hard cutover process, including the pre-cutover state, the planned outage window during activation, and the final post-cutover production state.

![Hard cutover migration workflow](/images/dl_migration-scenario_2.svg){: caption="Hard cutover migration workflow" caption-side="bottom"}

The specific steps vary depending on your network configuration, such as whether the two links must share the same physical resources (interfaces or IP addresses) or whether separate resources are available and a hard cutover is preferred. Review and adapt these steps to your specific scenario well in advance of the actual migration.
{: important}

### Assumptions
{: #hard-cutover-assumptions}

Before you start the migration, make sure that the following prerequisites are in place to support a controlled and predictable cutover process.

- A planned outage window is acceptable for the cutover.
- BGP is configured for both {{site.data.keyword.dl_short}} connections.
- You have administrative access to both your on-premises router (CE) and your {{site.data.keyword.cloud_notm}} account.
- Coordination with your telco/NSP partner is established.

### Phase 1: Preparation and provisioning
{: #hard-cutover-phase-1}

In this phase, you prepare the new {{site.data.keyword.dl_short}} environment by provisioning resources and establishing the required physical and logical connectivity in advance of the cutover.

This phase is largely the same as the [graceful traffic shift migration](#graceful-phase-1).
{: note}

#### Order and provision Direct Link
{: #order-provision-dl2-hard}

Order and set up the new {{site.data.keyword.dl_short}} instance in {{site.data.keyword.cloud_notm}} and coordinate circuit provisioning with your network provider.

1. Order and provision the new {{site.data.keyword.dl_short}} instance (Dedicated or Connect, Location, Speed, Partner) in {{site.data.keyword.cloud_notm}}. For detailed ordering instructions:

   - For {{site.data.keyword.dl_short}} Dedicated, see [Ordering {{site.data.keyword.dl_short}} Dedicated](/docs/dl?group=ordering-ibm-cloud-direct-link-dedicated).
   - For {{site.data.keyword.dl_short}} Connect, see [Ordering {{site.data.keyword.dl_short}} Connect](/docs/dl?group=ordering-ibm-cloud-direct-link-connect).

      Connect orders might require the use of your provider's portal for provisioning.
      {: note}

1. Work with your telco/NSP partner to establish the circuit.

#### Configure physical and logical connectivity
{: #configure-connectivity-hard}

Next, configure the physical interface on your on-premises router (IP address and MTU). Consider the use of jumbo frames (MTU 9000) for improved performance if supported by your network path and {{site.data.keyword.cloud_notm}} configuration. Make sure to establish L2/L3 connectivity to the NSP demarcation.

#### Configure the Direct Link gateway
{: #configure-dl2-gateway-hard}

After connectivity is in place, configure the {{site.data.keyword.dl_short}} gateway details (BGP ASN, Peering IPs, Authentication). Choose your connection type:

- Attach the necessary VPCs or Classic Infrastructure. They can be the same resources that are attached to {{site.data.keyword.dlc_short}}.
- Connect to a {{site.data.keyword.tg_short}} for centralized connectivity to multiple network types (VPCs, Classic, GRE tunnels, {{site.data.keyword.powerSys_notm}}).

#### Configure on-premises router interface
{: #configure-onprem-interface}

Finally, configure the corresponding interface IP address and MTU. Consider the use of jumbo frames (MTU 9000) for improved performance if supported by your network path and {{site.data.keyword.cloud_notm}} configuration.

### Phase 2: Pre-cutover configuration and verification
{: #hard-cutover-phase-2}

In this phase, prepare and validate the {{site.data.keyword.dl_short}} connection and make sure it does not affect production traffic, so you can safely proceed toward the final cutover.

If your router has available ports and the links do not share the same physical path or IP addresses, you can keep both connections active during this phase. Apply restrictive BGP policies to prevent the {{site.data.keyword.dl_short}} connection from affecting production traffic.

If the links share the same resources, configure and test the new link during a separate maintenance window before the final cutover. Or you can proceed directly to [Phase 3](#hard-cutover-phase-3) after you complete Phase 1.
{: note}

#### Configure Direct link BGP (no production impact)
{: #configure-dl2-bgp-hard}

Set up BGP for the {{site.data.keyword.dl_short}} connection by using restrictive routing policies to help ensure it is fully configured but isolated from production traffic.

1. Define the BGP neighbor relationship toward the {{site.data.keyword.cloud_notm}} peer IPs for {{site.data.keyword.dl_short}} (Customer ASN, Peer ASN, MD5 key if used).
1. Apply restrictive BGP policies to prevent this session from impacting production traffic before the cutover:

**Outbound policy (On-Premises → IBM):**

Configure route maps to block advertisement of all your production on-premises prefixes toward the {{site.data.keyword.dl_short}} peer. You might permit only a specific, nonproduction test prefix if needed for verification.

{{site.data.keyword.dl_short}} provides route filtering capabilities that can help control route advertisements. For more information, see [Setting up route filters](/docs/dl?group=setting-up-route-filters).
{: tip}

**Inbound policy (IBM → On-Premises):**

Configure route maps to reject routes received from the {{site.data.keyword.dl_short}} peer, or accept them but set a minimal `local-preference`, for example, `0` or `1`, which helps make sure that they are never preferred while {{site.data.keyword.dlc_short}} is active, or filter them such that they aren't installed in the main routing table, for example, by using VRFs for testing if applicable. The goal is zero impact on the production routing table from this BGP session initially.

#### Establish and verify BGP session
{: #establish-bgp-session-hard}

Bring up the BGP session and confirm it is established. Make sure the old {{site.data.keyword.dlc_short}} continues to carry all production traffic.

1. Enable the BGP neighbor configuration for {{site.data.keyword.dl_short}}. The restrictive policies that you configured help make sure that the configuration does not impact production routes.
1. Verify that the BGP session state becomes `Established` by checking the BGP summary on your router. If both links can be connected simultaneously, the BGP session is established while {{site.data.keyword.dlc_short}} remains active and carrying production traffic.
1. Check the router logs for errors.

#### Verify connectivity (nonproduction)
{: #verify-connectivity-hard}

Validate basic connectivity and performance over the {{site.data.keyword.dl_short}} path by using controlled or test traffic only.

1. Run `pings` and `traceroutes` that are sourced from the {{site.data.keyword.dl_short}} interface IP on your router toward the {{site.data.keyword.cloud_notm}} BGP peer IP.
1. If you advertised a test prefix and allowed a test {{site.data.keyword.cloud_notm}} prefix in, run end-to-end tests for that specific test path to confirm basic data plane forwarding over {{site.data.keyword.dl_short}}.
1. Consider running performance tests by using your preferred tools such as `iperf`, `netperf`, or other bandwidth and latency testing tools to validate throughput and latency characteristics of the new {{site.data.keyword.dl_short}} path.

At this point, {{site.data.keyword.dl_short}} is fully provisioned, the interfaces are up, and BGP and basic connectivity have been verified. However, it is not yet ready for production routing or changes to the main routing table. {{site.data.keyword.dlc_short}} continues to carry all production traffic.
{: important}

### Phase 3: Hard cutover execution (planned outage window)
{: #hard-cutover-phase-3}

During this phase, traffic is switched from the old {{site.data.keyword.dlc_short}} connection to {{site.data.keyword.dl_short}} within a planned maintenance window, resulting in a temporary service interruption while routing is updated.

**[START of Outage Window]**{: tag-red}

#### Disable old link (Direct Link on Classic)
{: #disable-old-link}

On your on-premises router, begin the cutover by disabling the old {{site.data.keyword.dlc_short}} connection. The method depends on your configuration:

- **If both links share the same physical interface or IP addresses**: Remove the old link's configuration and apply the new link's configuration to the same interface. The switch is a definitive cut.

- **If the links are on separate interfaces**: Administratively shut down the BGP neighbor session for {{site.data.keyword.dlc_short}}, or shut down the physical interface.

Verify that the old link's interface or BGP session goes down. Production traffic that traverses {{site.data.keyword.dlc_short}} now drops, creating the outage.

#### Enable new Direct Link for production
{: #enable-new-link}

Activate production routing over {{site.data.keyword.dl_short}} by updating BGP policies to advertise and accept production traffic.

1. On your on-premises router, immediately modify the BGP policies that are applied to the {{site.data.keyword.dl_short}} neighbor:
   * **Outbound Policy**: Change the route map to allow advertisement of all necessary production on-premises prefixes toward {{site.data.keyword.cloud_notm}}.
   * **Inbound Policy**: Change the route map to accept routes that are received from {{site.data.keyword.cloud_notm}} and assign them a standard `local-preference` (for example, `100`), allowing them to be installed as best paths in the routing table.
1. Apply the policy changes.

#### Confirm BGP convergence
{: #confirm-bgp-convergence}

Verify that BGP fully converges and that routing stabilizes on the {{site.data.keyword.dl_short}} connection.

1. Reset the BGP session for {{site.data.keyword.dl_short}} to force immediate prefix readvertisement and processing with the new policies. Use a soft reset if your router supports it; otherwise, perform a hard reset of the BGP session.

1. Monitor the BGP session state and help ensure that it stays or becomes `Established`.

### Phase 4: Post-cutover verification and monitoring
{: #hard-cutover-phase-4}

After the cutover is complete, validate routing, connectivity, and application behavior to help ensure that traffic is successfully and stably running over {{site.data.keyword.dl_short}}.

#### Verify routing
{: #verify-routing-hard}

Confirm that routing converges correctly and that {{site.data.keyword.dl_short}} is now being used as expected for all relevant traffic flows.

1. Check routes that are advertised to and received from the {{site.data.keyword.dl_short}} BGP neighbor. Confirm that they match the new policies.

1. Check the BGP table and the main IP routing table. Confirm that routes toward {{site.data.keyword.cloud_notm}} networks now use the {{site.data.keyword.dl_short}} next-hop IP address and that your on-premises prefixes are being advertised correctly.

#### Test connectivity and applications
{: #test-connectivity-hard}

Test end-to-end connectivity and your applications to help ensure services function correctly over the new {{site.data.keyword.dl_short}} connection.

1. Immediately run end-to-end connectivity tests (`ping`, `traceroute`) from on-premises to {{site.data.keyword.cloud_notm}} services and vice versa.
1. Have application teams test critical applications that rely on this connectivity.

#### Monitor performance
{: #monitor-performance-hard}

Continuously monitor traffic, latency, and application performance to confirm stable operation after the migration.

1. Closely monitor traffic rates (Tx and Rx) on the {{site.data.keyword.dl_short}} interface.
1. Monitor latency, packet loss (if any), and application performance metrics.

**[END of Outage Window]**{: tag-red} (after verification is successful)

### Phase 5: Decommissioning Direct Link on Classic
{: #hard-cutover-phase-5}

After the cutover is complete and validated, fully retire the legacy {{site.data.keyword.dlc_short}} environment to avoid unnecessary cost and configuration drift.

#### Stabilization period
{: #stabilization-period-hard}

Before you remove any legacy components, monitor the stability of the {{site.data.keyword.dl_short}} link for a suitable period.

#### Remove Direct Link on Classic configuration
{: #remove-dl1-config-hard}

After stability is confirmed, remove the BGP configuration, route maps, and interface configuration related to {{site.data.keyword.dlc_short}} from your on-premises router.

#### Deprovision Direct Link on Classic
{: #deprovision-dl1-hard}

Formally decommission the legacy service and associated circuit.

1. Delete the {{site.data.keyword.dlc_short}} gateway in the {{site.data.keyword.cloud_notm}} console.
1. Request formal decommissioning of the circuit from your telco/NSP partner.

#### Update documentation
{: #update-documentation-hard}

Update all network diagrams and documentation.

## Important considerations
{: #important-considerations}

Keep the following key considerations in mind throughout the migration process to reduce risk and help ensure a smooth transition:

Planning
:   Plan this migration carefully, including IP addressing, BGP policy details, and rollback steps.

Maintenance window
:   Run the traffic-shifting steps (Phase 3) during a planned maintenance window to minimize business impact.

Testing
:   Thoroughly test the {{site.data.keyword.dl_short}} path before you shift production traffic.

Monitoring
:   Have robust monitoring in place (interface stats, BGP states, latency, application metrics) throughout the process.

Rollback plan
:   Know how to quickly revert BGP policy changes (for example, revert `local-preference` and `AS_PATH` prepending) to shift traffic back to {{site.data.keyword.dlc_short}} if issues arise during the migration.

Stakeholder communication
:   Keep your telco/NSP partner and relevant {{site.data.keyword.cloud_notm}} support teams informed about your plans and schedule.

Firewalls and NAT
:   If you have firewalls or NAT devices inline, make sure that their rules and translations are updated or duplicated for the new {{site.data.keyword.dl_short}} path IPs and interfaces.

## Key differences between migration scenarios
{: #scenario-differences}

The following table highlights the tradeoffs between graceful migration and hard cutover approaches to help you choose the best option for your environment:

| Aspect | Graceful migration | Hard cutover |
| ------ | ------------------ | ------------ |
| Outage required | No planned outage | Planned service interruption (typically 15 to 30 minutes) |
| Complexity | More complex BGP attributes tuning | Simpler cutover actions (disable old, enable new) |
| Risk | Lower risk because you can verify the new link before full cutover | Higher risk if the new link fails during cutover |
| Rollback | Revert by adjusting BGP policies | Requires re-enabling the old link, which can extend the outage |
| Testing | Test with production traffic gradually | Limited testing before cutover |
| Typical duration | 2 to 4 hours for traffic shift | 15 to 30-minute outage window |
| Best for | Production environments that require minimal disruption | Environments where parallel paths are not possible |
{: caption="Comparison of migration scenarios" caption-side="bottom"}

## Migration timeline
{: #migration-timeline}

Use the following estimated timelines to plan and coordinate your migration activities.

### Graceful migration timeline
{: #graceful-timeline}

This approach spreads changes over time, allowing validation at each stage with minimal disruption.

| Phase | Estimated duration | Can be done during business hours? |
| ----- | ------------------ | ---------------------------------- |
| Phase 1: Preparation and provisioning | 1 to 2 weeks | Yes |
| Phase 2: Parallel path setup and verification | 2 to 4 hours | Yes |
| Phase 3: Traffic migration | 2 to 4 hours | Recommended during a maintenance window |
| Phase 4: Decommissioning | 1 to 2 weeks | Yes |
{: caption="Graceful migration timeline" caption-side="bottom"}

### Hard cutover timeline
{: #hard-cutover-timeline}

This approach compresses changes into a shorter window but requires a planned outage.

| Phase | Estimated duration | Can be done during business hours? |
| ----- | ------------------ | ---------------------------------- |
| Phase 1: Preparation and provisioning | 1 to 2 weeks | Yes |
| Phase 2: Pre-cutover configuration | 2 to 4 hours | Yes |
| Phase 3: Cutover execution | 15 to 30 minutes | No - requires a maintenance window |
| Phase 4: Post-cutover validation | 1 to 2 hours | No - during the maintenance window |
| Phase 5: Decommissioning | 1 to 2 weeks | Yes |
{: caption="Hard cutover timeline" caption-side="bottom"}

## Migration checklists
{: #migration-checklists}

Use these checklists to track your progress and make sure that no critical steps are missed during the migration.

### Pre-migration checklist
{: #pre-migration-checklist}

Complete the following tasks before you begin the migration:

- [ ] {{site.data.keyword.dl_short}} ordered and provisioned
- [ ] Physical connectivity that is established and tested
- [ ] BGP peering configuration prepared
- [ ] Route policies and filters documented
- [ ] Monitoring tools that are configured and ready
- [ ] Rollback plan that is documented and reviewed
- [ ] Stakeholders notified of migration schedule
- [ ] Maintenance window scheduled (for hard cutover)

### During migration checklist
{: #during-migration-checklist}

Validate each step as you progress through the migration:

- [ ] BGP session established successfully
- [ ] Routes exchanged and verified
- [ ] Connectivity tests passed
- [ ] Traffic that is shifted to new link
- [ ] Performance metrics validated
- [ ] Applications that are tested and verified
- [ ] No errors in router logs

### Post-migration checklist
{: #post-migration-checklist}

After migration, confirm that cleanup and stabilization tasks are complete:

- [ ] Stabilization period completed (24 to 48 hours minimum)
- [ ] {{site.data.keyword.dlc_short}} link decommissioned
- [ ] Router configuration cleaned up
- [ ] Network documentation updated
- [ ] Network diagrams updated
- [ ] Monitoring baselines adjusted

## Troubleshooting
{: #troubleshooting-dl}

If you encounter issues during or after migration, use the following guidance to identify and resolve common problems.

### BGP session fails to establish
{: #bgp-session-issues}

If your BGP session fails to establish:

1. Verify IP addressing matches on both sides of the connection.
1. Check the MD5 authentication key if configured (case-sensitive).
1. Verify that firewall rules allow BGP traffic (TCP port `179`).
1. Check for MTU mismatches between interfaces.
1. Verify that the correct BGP ASN is configured on both sides.
1. Check router logs for specific BGP error messages.

### Traffic not shifting as expected
{: #traffic-shift-issues}

If traffic doesn't shift after policy changes:

1. Verify that BGP policies are applied to the correct neighbor.
1. Perform a soft reset of the BGP session to apply policy changes (both inbound and outbound directions).
1. Check that the BGP table shows the expected best paths with correct attributes.
1. Verify that the routing table reflects BGP changes.
1. Check for route filtering that might be blocking prefixes.
1. Verify `local-preference` and `AS_PATH` values are as expected.

### Routes are not being advertised
{: #route-advertisement-issues}

If routes are not being advertised as expected:

1. Check the outbound route-map or filter-list configuration.
1. Verify network statements in BGP configuration.
1. Check for prefix-list or access-list blocking routes.
1. Verify that routes exist in the routing table before BGP advertisement.
1. Check for maximum-prefix limits that might be blocking advertisements.

### Performance degradation after migration
{: #performance-issues}

If you experience performance issues after migration:

1. Check interface statistics for errors or drops.
1. Verify MTU settings match across the path.
1. Check for asymmetric routing issues.
1. Monitor latency and jitter metrics.
1. Verify Quality of Service (QoS) policies are applied correctly.
1. Check for congestion on the new link.

For more troubleshooting guidance and solutions for {{site.data.keyword.dl_short}}, see [Troubleshooting {{site.data.keyword.dl_short}}](/docs/dl?group=troubleshooting).
{: tip}

## Next steps
{: #next-steps}

After you complete your {{site.data.keyword.dl_short}} migration, take the following actions to optimize and maintain your new environment:

- Monitor the new {{site.data.keyword.dl_short}} connection for stability.
- Update network documentation and diagrams.
- Review and optimize BGP policies.
- Consider implementing redundancy with multiple {{site.data.keyword.dl_short}} connections.
- Set up monitoring and alerting for the new connection.
- Evaluate whether {{site.data.keyword.tg_short}} connectivity can simplify your network architecture by providing centralized routing to multiple network types.

## Related links
{: #related-links}

- [{{site.data.keyword.dl_short}} documentation](/docs/dl)
- [BGP best practices](/docs/dl?topic=dl-models-for-diversity-and-redundancy-in-direct-link)
- [Troubleshooting {{site.data.keyword.dl_short}}](/docs/dl?group=troubleshooting)
- [{{site.data.keyword.dl_short}} pricing and billing](/docs/dl?topic=dl-faqs#faq-pricing-billing)
- [High availability and disaster recovery](/docs/dl?topic=dl-ha-dr-dl)
- [{{site.data.keyword.tg_short}} documentation](/docs/transit-gateway)
- [Connecting {{site.data.keyword.dl_short}} to {{site.data.keyword.tg_short}}](/docs/transit-gateway?topic=transit-gateway-helpful-tips#dl_considerations)
- [Classic to VPC migration overview](/docs/classic-to-vpc?topic=classic-to-vpc-about-migration-infra)
- [{{site.data.keyword.cloud_notm}} Architecture Center](https://www.ibm.com/think/architectures/patterns){: external}
