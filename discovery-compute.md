---

copyright:
  years: 2025
lastupdated: "2025-11-07"

keywords: compute, classic VSI

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}


# Discovery of Classic Compute Resources
{: #discovery-of-classic-compute-resources}

Before migrating from Classic Infrastructure to VPC Infrastructure, it is important to inventory all existing Classic resources and capture details about their configuration and billing. This ensures you can plan capacity, cost, and migration approach accurately.
{: shortdesc}


## Information to Collect
{: #compute-information-to-collect}

When preparing for migration, gather the following information for each Classic VSI:

- **vCPUs and Memory**: Number of CPUs and amount of RAM  
- **Location**: Datacenter in which the VSI is provisioned  
- **VSI Type**: Public, Dedicated, Private, Variable, or Transient  
- **Attached Storage**: Disk sizes, types (SAN, local), and number of volumes  
- **Operating System**: OS type and version  
- **Billing Model**: Hourly or Monthly  
- **GPUs**: Count and type (if provisioned)  
- **Software Add-ons**: Any additional software or licenses installed through IBM Cloud (e.g., cPanel, Plesk, antivirus)  
- **IP Addresses & VLANs**: Take note if the virtual server is only using a private network and IP has both a public and private network
- **Placement Groups**: Note if the VSI is part of a placement group and capture the placement policy
- **Security Groups**: Document the firewall rules, inbound/outbound traffic policies, and group membership applied to the VSI
- **Reserved Instance Term**: 1-year or 3-year commitment (if applicable)  
- **Dedicated Hosts**: If the VSI is provisioned on a dedicated host, record the host ID, host group, and placement policy

This data allows you to map existing Classic workloads to equivalent **VPC profiles**, and to plan billing alignment.  

---

## Using IBM Cloud CLI 
{: #using-cloud-cli}

The `ibmcloud sl` CLI plugin allows you to query Classic Infrastructure resources.

### List All Virtual Servers with CPU, Memory, IP Addresses, Datacenter
{: #list-all-virtual-servers-cli-example}

```bash
ibmcloud sl vs list
```

Example output:

```bash
id        hostname      domain        cpu   memory  public_ip      private_ip     datacenter  action
12345678  web-server-1  mydomain.com  4     8192    169.45.123.45  10.120.123.45  dal10     
87654321  db-server-1   mydomain.com  8     16384   -              10.123.222.11  wdc07     
```

#### Show Full Details for a VSI
{: #show-full-details-for-vsi-cli-example}

```bash
ibmcloud sl vs detail 12345678
```

This returns detailed information including attached storage, vlans and IP addresses, security groups, and VSI type.

### Using IBM Cloud CLI to List Classic Dedicated Hosts
{: #cli-to-list-classic-dedicatedhosts}

```bash
ibmcloud sl dedicatedhost list
```

Example output:

```bash
Id       Name          Datacenter   Router         Cpu (allocated/total)   Memory (allocated/total)   Disk (allocated/total)   Guests
791431   web-servers   dal10        bcr01a.dal13   4/56                    8/242                      100/1200                   1
791531   db-servers    wdc07        bcr01a.wdc07   8/56                    16/242                     100/1200                   1
```

#### List VSIs on a Dedicated Host
{: #list-vsis-on-a-dedicated-host}

```bash
ibmcloud sl dedicatedhost list-guests 791431
```

This returns a list of VSIs that are provisioned on the dedicated host.

---

## Using REST API 
{: #using-rest-api}

### Prerequisites 
{: #rest-api-prerequisites}

You’ll need the following:

- **SoftLayer API credentials** (username and API key)  
   > To get started with SoftLayer API credentials, follow the official IBM Cloud guide:  
   > [Managing classic infrastructure API keys](/docs/account?topic=account-classic_keys)
- **`curl`** installed on your machine
- **`jq`** for parsing and masking JSON output

### API Endpoint
{: #rest-api-endpoint}

SoftLayer REST API base URL:

```bash
https://api.softlayer.com/rest/v3.1
```

The endpoint for listing virtual guests:

```bash
https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getVirtualGuests
```

#### API Retrieve Virtual Guests
{: #rest-retrieve-virtual-guests}

Run the following curl command to retrieve virtual guest details:

```bash
curl -u "${USER}:${API_KEY}" -g \
"https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getVirtualGuests?objectMask=mask[id,hostname,maxCpu,maxMemory,type[name],blockDevices[device,diskImage[capacity]],placementGroup[name],networkComponents[id,primaryIpAddress,networkVlan[vlanNumber,name],securityGroupBindings[id,securityGroup[name]]],softwareComponents[id,softwareDescription[manufacturer,version,name]],gpuCount,gpuType,dedicatedHost[name,cpuCount,memoryCapacity,diskCapacity]]" \
| jq .
```

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
