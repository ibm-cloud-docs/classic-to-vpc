---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: compute, classic virtual server instance

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Discovery of classic compute resources
{: #discover-classic-compute-resources}

Before you migrate from classic infrastructure to a {{site.data.keyword.vpc_full}} (VPC) infrastructure, it is important to create an inventory of all existing classic resources and capture their configuration and billing details. This approach helps ensure that you can plan capacity, cost, and migration approach accurately.
{: shortdesc}

## Information to collect
{: #compute-information-to-collect}

When you prepare for migration, gather the following information for each classic virtual server instance:

- vCPUs and memory: Number of CPUs and amount of RAM.
- Location: Data center in which the virtual server instance is provisioned.
- Virtual server instance type: Public, dedicated, private, variable, or transient.
- Attached storage: Disk sizes, types (SAN, local), and number of volumes.
- Operating system (OS): OS type and version.
- Billing model: Hourly or monthly.
- GPUs: The count and the type (if provisioned).
- Software add-ons: Any additional software or licenses that are installed through {{site.data.keyword.cloud}} (for example, cPanel, Plesk, antivirus).
- IP addresses and VLANs: Check whether the virtual server uses only a private network and whether the virtual server has both a public and private network.
- Placement groups: Check whether the virtual server instance is part of a placement group and captures the placement policy.

### Additional validation steps
{: #validation-steps}

- Security groups: Document the firewall rules, inbound and outbound traffic policies, and group membership applied to the virtual server instance.
- Reserved instance term: Specify whether a 1-year or 3-year commitment applies.
- Dedicated hosts: If the virtual server instance is provisioned on a dedicated host, record the host ID, host group, and placement policy.

This data helps you to map existing classic workloads to equivalent {{site.data.keyword.vpc_short}} profiles and plan billing alignment.

## From the IBM Cloud CLI
{: #using-cloud-cli}
{: cli}

With the `ibmcloud sl` CLI plug-in, you can query classic infrastructure resources.

### Listng all virtual servers with CPU, memory, IP addresses, datacenter
{: #list-all-virtual-servers-cli-example}

```sh
ibmcloud sl vs list
```
{: pre}

Example output:

```sh
id        hostname      domain        cpu   memory  public_ip      private_ip     datacenter  action
12345678  web-server-1  mydomain.com  4     8192    169.45.123.45  10.120.123.45  dal10
87654321  db-server-1   mydomain.com  8     16384   -              10.123.222.11  wdc07
```
{: screen}

#### Showing the full details for a virtual server instance
{: #show-full-details-for-vsi-cli-example}

```sh
ibmcloud sl vs detail 12345678
```
{: pre}

Running this command returns detailed information, including attached storage, VLANs, IP addresses, security groups, and the virtual server instance type.

### Listing classic dedicated hosts
{: #cli-to-list-classic-dedicatedhosts}

```sh
ibmcloud sl dedicatedhost list
```
{: pre}

Example output:

```sh
Id       Name          Datacenter   Router         Cpu (allocated/total)   Memory (allocated/total)   Disk (allocated/total)   Guests
791431   web-servers   dal10        bcr01a.dal13   4/56                    8/242                      100/1200                   1
791531   db-servers    wdc07        bcr01a.wdc07   8/56                    16/242                     100/1200                   1
```
{: screen}

#### Listing virtual server instances on a dedicated host
{: #listing-vsis-on-a-dedicated-host}

```sh
ibmcloud sl dedicatedhost list-guests 791431
```
{: pre}

This command returns a list of virtual server instances that are provisioned on the dedicated host.

## With the REST API
{: #using-rest-api}
{: api}

### Prerequisites
{: #rest-api-prerequisites}

You need to meet the following requirements:

- SoftLayer API credentials (username and API key).
  - To get started with SoftLayer API credentials, follow the official {{site.data.keyword.cloud_notm}} guide to [Managing classic infrastructure API keys](/docs/account?topic=account-classic_keys).
- Install `curl` on your system.
- Use `jq` for parsing and masking JSON output.

### API endpoint
{: #rest-api-endpoint}

SoftLayer REST API base URL is:

```sh
https://api.softlayer.com/rest/v3.1
```
{: pre}

The endpoint for listing virtual guests:

```sh
https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getVirtualGuests
```
{: pre}

#### Listing the virtual guests with the API
{: #rest-retrieve-virtual-guests}

Run the following curl command to retrieve virtual guest details:

```sh
curl -u "${USER}:${API_KEY}" -g \
"https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getVirtualGuests?objectMask=mask[id,hostname,maxCpu,maxMemory,type[name],blockDevices[device,diskImage[capacity]],placementGroup[name],networkComponents[id,primaryIpAddress,networkVlan[vlanNumber,name],securityGroupBindings[id,securityGroup[name]]],softwareComponents[id,softwareDescription[manufacturer,version,name]],gpuCount,gpuType,dedicatedHost[name,cpuCount,memoryCapacity,diskCapacity]]" \
| jq .
```
{: pre}

Example Response

```json
[
  {
    "hostname": "webserver-1",
    "id": 999999,
    "maxCpu": 2,
    "maxMemory": 4096,
    "blockDevices": [
      { "device": "0", "diskImage": { "capacity": 25 } },
      { "device": "1", "diskImage": { "capacity": 2 } }
      { "device": "2", "diskImage": { "capacity": 100 } }
    ],
    "networkComponents": [
      {
        "id": 888888,
        "networkVlan": { "vlanNumber": 892 },
        "primaryIpAddress": "10.186.92.239",
        "securityGroupBindings": []
      },
      {
        "id": 777777,
        "securityGroupBindings": []
      }
    ],
    "placementGroup": { "name": "webserver-group" },
    "softwareComponents": [
      {
        "id": 333333,
        "softwareDescription": {
          "manufacturer": "Redhat",
          "name": "EL",
          "version": "9.0-64 Minimal for VSI"
        }
      },
      {
        "id": 222222,
        "softwareDescription": {
          "manufacturer": "MySQL",
          "name": "MySQL (Linux Only)",
          "version": "Latest"
        }
      }
    ],
    "type": { "name": "Public" }
  }
]
```
{: codeblock}
