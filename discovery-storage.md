---

copyright:
  years: 2025
lastupdated: "2025-11-19"

keywords: storage, classic VSI

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}


# Discovery of Classic Storage Resources
{: #discovery-of-classic-storage-resources}

IBM Cloud Classic Infra offers following storage options -

**Local storage:** Local storage is built on disks that are local to the virtual server host. Local storage provides improved disk read and write performance. For more information about available data centers for local SSD storage, see [Balanced local storage](/docs/virtual-servers?topic=virtual-servers-about-virtual-server-profiles#balanced-local-storage).

**Block Storage:** IBM Cloud® Block Storage for Classic is persistent, high-performance iSCSI storage that is provisioned and managed independently of Compute instances. For more information on provisioning , see [Block Storage provisioning](/docs/BlockStorage?group=ordering-storage)  

**File Storage:** IBM Cloud® File Storage for Classic is persistent, fast, and flexible network-attached, NFS-based File Storage for Classic. For more information about provisioning File share, see [File Storage provisioning](/docs/FileStorage?group=ordering-storage) 

**Portable SAN Storage:** Portable storage volumes are auxiliary storage solutions that are exclusively available on Virtual Servers. For more information about attaching portable storage see [attach portable storage](/docs/virtual-servers?topic=virtual-servers-accessing-portable-storage#attaching-portable-storage)

## Information to Collect
{: #storage-information-to-collect}

When preparing for migration, gather the following information for each Classic Storage:

- **Storage type**: Block Storage, File Storage, Portable Storage and Local Storage.
- **Profile type**: Endurance and Performance type for Block and File.
- **Volume capacity**: Size of the storage volume attached to the device.
- **Volume iops**: IOPs of the storage volume attached to the device.
- **NFS usage**: Size of the storage volume used in the device (Applicable for NFS shares only).
- **Snapshots**: Snapshot details for Block and File Storage.
- **Replica Volume**: Replica details for Block and File Storage.


The `ibmcloud sl` CLI plugin allows you to query Classic Infrastructure resources.

### List All Storage attached to Virtual Servers 
{: #list-all-storage-attached-to-virtual-servers-cli-example}

```bash
ibmcloud sl vs storage <vs_id>
```

Example output:

```bash
$ ibmcloud sl vs storage 153017465


Block Storage Details
iSCSI
Username                    Password           IQN
IBM02SU1041833-V153017465   *********   iqn.2004-10.com.ubuntu:01:V153017465 
           capacity   Target address   Location   Notes
LUN name

Portable Storage
Description                     Capacity   Location
sach-virutalserver01 - Disk 2   10         Dallas 10

File Storage Details
Volume name           capacity   Hostname                               Location    Notes
IBM02SEV1041833_858   20         fsf-dal1003i-fz.adn.networklayer.com   Dallas 10     Automation Storage Test - SMT - MOUNT ""

System storage details
Type     Name   Drive   Capacity
System   Disk   0       25 GB
Swap     Disk   1       2 GB
System   Disk   2       10 GB
System   Disk   7       64 MB

```

### List All Storage attached to Bare metal 
{: #list-all-storage-attached-to-bare-metal-cli-example}

```bash
ibmcloud sl hardware storage <bm_id>
```

Example output:

```bash
Block Storage Details
iSCSI
Username                  Password           IQN
IBM02SU1041833-H3481996   ***************   iqn.2004-10.com.ubuntu:01:H3481996 
           capacity   Target address   Location   Notes
LUN name

File Storage Details
Volume name   capacity   Hostname   Location   Notes

Other storage details
Type         Name                                      Capacity    Serial #
Hard Drive   Micron 7450 PRO MTFDKBA480TFR-1BC15ABYY   480.00 GB   241448485d08
Hard Drive   Micron 7450 PRO MTFDKBA480TFR-1BC15ABYY   480.00 GB   241448485c1f
Hard Drive   Intel S4610 Series                        960.00 GB   phyg14050010960cgn
```


---


## Discover Block Storage
{: #discover-block-storage}

Block Storage for Classic brings best-in-class levels of durability and availability with an unmatched feature set. It is built by using industry standards and best practices.  Block Storage for Classic is designed to protect the integrity of the data and maintain availability through maintenance events and unplanned failures, and provide a consistent performance baseline.

For more information see [Getting started with Block Storage for Classic](/docs/BlockStorage?topic=BlockStorage-getting-started)

### List All iSCSI Block Storage Volumes in the account
{: #list-all-block-storage-volumes-in-the-account-example}

Using CLI -

```bash
ibmcloud sl block volume-list
```

Example output:

```bash
id          username                    datacenter   storage_type                          capacity_gb   bytes_used   IOPs   ip_addr          lunId   active_transactions   rep_partner_count   notes
195144982   SL02SL1041833_63_REP_1      -            performance_block_storage_replicant   20            -            100    192.0.2.10     -       0                     0                   -
217954031   SL02SEL1041833-537          dal10        endurance_block_storage               35            -            -      192.0.2.11    25      0                     0                   DISASTER RECOVERY FAILOVER COMPLETED
227445958   SL02SEL1041833-512          dal10        endurance_block_storage               100           -            -      192.0.2.12    7       0                     0                   DISASTER RECOVERY FAILOVER COMPLETED + FAILBACK COMPLETED
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Account/{SoftLayer_AccountID}/getIscsiNetworkStorage'
```
For more information, see [List all block volumes in the account](https://sldn.softlayer.com/reference/services/SoftLayer_Account/getIscsiNetworkStorage/)

### Show Full Details for a iSCSI Block Volume
{: #show-full-details-for-block-volume-example}

Using CLI -

```bash
ibmcloud sl block volume-detail <block_vol_id>
```

Example output:

```bash
$ ibmcloud sl block volume-detail 721685964
Name                       Value
ID                         721685964
User name                  IBM02SEL1041833-888
Type                       endurance_block_storage
Capacity (GB)              20
LUN Id                     0
Endurance Tier             LOW_INTENSITY_TIER
Endurance Tier Per IOPS    0.25
Datacenter                 dal13
Target IP                  198.51.100.42
# of Active Transactions   0
Replicant Count            0
Notes                      -
Encrypted                  True
```

Using API -


```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{Block_Vol_ID}?objectMask=mask[id,username,password,capacityGb,snapshotCapacityGb,parentVolume.snapshotSizeBytes,storageType.keyName,serviceResource.datacenter.name,serviceResourceBackendIpAddress,storageTierLevel,iops,lunId,originalVolumeName,originalSnapshotName,originalVolumeSize,hasEncryptionAtRest,activeTransactionCount,activeTransactions.transactionStatus.friendlyName,replicationPartnerCount,replicationStatus,replicationPartners.id,replicationPartners.username,replicationPartners.serviceResourceBackendIpAddress,replicationPartners.serviceResource.datacenter.name,replicationPartners.replicationSchedule.type.keyname,notes]'
```

For more information, see [get Block volume details](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/)


### Show Snapshot Details for a iSCSI Block Volume
{: #show-snapshot-details-for-block-volume-example}

Using CLI -

```bash
ibmcloud sl block snapshot-list <block_vol_id>
```

Example output:

```bash
ibmcloud sl block snapshot-list 721685964               
id          user_name              created                     size_bytes   notes
722754994   IBM02SEVC1041833_943   2025-10-28T01:20:00-05:00   163840       -
```

Using API - 

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{Block_Vol_ID}/getSnapshots?objectMask=mask[id,username,notes,snapshotSizeBytes,storageType.keyName,snapshotCreationTimestamp,hourlySchedule,dailySchedule,weeklySchedule]'
```

For more information, see [List snapshots for iSCSI block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getSnapshots/)

### Show Replica Details for a iSCSI Block Volume
{: #show-replica-details-for-block-volume-example}

Using CLI -

```bash
ibmcloud sl block replica-partners <block_vol_id>
```
Example output:

```bash
ibmcloud sl block replica-partners 721685964
ID          User name                   Account ID   Capacity (GB)   Hardware ID   Guest ID   Host ID
721870496   IBM02SEL1041833_888_REP_1   1041833      20              -             -          -
```

Using API - 
```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{SoftLayer_Network_Storage_IscsiID}/getReplicationPartners'
```
For more information , see [getReplicationPartners for iSCSI block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getReplicationPartners/)


### Find mount path in VSI using iSCSI Block Volume details
{: #find-mount-path-in-vsi-using-block-volume-details-example}

1) Get VSI details that is authorized to use iSCSI Block volume

Using CLI - 

```bash
ibmcloud sl block access-list <block_vol_id>
```

Example output:

```bash
$ ibmcloud sl block access-list 721685964      

id          name                                  type      private_ip_address   source_subnet   host_iqn                                        username                    password           allowed_host_id
153885696   poc-classic-to-vpc.ibmcloud.private   VIRTUAL   203.0.113.99       -               iqn.2004-10.com.ubuntu:01:V153885696 
   IBM02SU1041833-V153885696   **********   2872634
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{SoftLayer_Network_Storage_IscsiID}/getAllowedVirtualGuests'
```
For more information, see [get authorized VS for a particular iSCSI Block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getAllowedVirtualGuests/)

Use 'ibmcloud sl vs detail <vsi_id>' to get more details on the VSI . ID of the VSI can be found under the first column in the o/p. In this case the id is '153885696'.

2) Get Target IP of the iSCSI block volume 

Refer Step 2 in [Find the target ip of the block volume](#show-full-details-for-block-volume-example)

3) Get the iscsi block device path 

Run 'iscsiadm -m session -P 3' within the VSI and map the Target IP obtained from Step 2 to the Current Portal ip in the o/p. Then check for 'Attached scsi disk' under 'Attached SCSI devices' to get the iscsi block device path on which the block volume is attached.

Example output:

```bash
$ iscsiadm -m session -P 3

iSCSI Transport Class version 2.0-870
version 2.1.9
Target: iqn.2004-10.com.ubuntu:01:60f3517884c3  (non-flash)
    Current Portal: 198.51.100.42:3260,1054
    Persistent Portal: 198.51.100.42:3260,1054
        **********
        Interface:
        **********
        Iface Name: default
        Iface Transport: tcp
        Iface Initiatorname: iqn.2004-10.com.ubuntu:01:60f3517884c3
        Iface IPaddress: 10.11.100.154
        Iface HWaddress: default
        Iface Netdev: default
        SID: 26
        iSCSI Connection State: LOGGED IN
        iSCSI Session State: LOGGED_IN
        Internal iscsid Session State: NO CHANGE
        *********
        Timeouts:
        *********
        Recovery Timeout: 5
        Target Reset Timeout: 30
        LUN Reset Timeout: 30
        Abort Timeout: 15
        *****
        CHAP:
        *****
        username: IBM02SU1041833-V153885696
        password: ********
        username_in: <empty>
        password_in: ********
        ************************
        Negotiated iSCSI params:
        ************************
        HeaderDigest: None
        DataDigest: None
        MaxRecvDataSegmentLength: 262144
        MaxXmitDataSegmentLength: 65536
        FirstBurstLength: 65536
        MaxBurstLength: 1048576
        ImmediateData: Yes
        InitialR2T: Yes
        MaxOutstandingR2T: 1
        ************************
        Attached SCSI devices:
        ************************
        Host Number: 2	State: running
        scsi2 Channel 00 Id 0 Lun: 0
            Attached scsi disk sda		State: running

```

4) Get mount path details

Run 'lsblk -f' command to get the mount path, file system type, uuid and available space of the file system mounted on the multipath device (In below case - sda and sdb are iscsi device paths on which ext4 file system is mounted on path - /mnt/mount1). The filesystem mount path should be used for migration of data to vpc. 

Example output: 

```bash
$ lsblk -f

NAME                                FSTYPE       FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0                                                                                                             0   100% /snap/core24/1225
loop1                                                                                                             0   100% /snap/slcli/2957
loop2                                                                                                             0   100% /snap/snapd/25577
sda                                 mpath_member                                                                           
└─3600a098038305659702b4f6f5a646f57 ext4         1.0                   b27cf189-9648-406d-a761-39274c1c142c   18.5G     0% /mnt/mount1
sdb                                 mpath_member                                                                           
└─3600a098038305659702b4f6f5a646f57 ext4         1.0                   b27cf189-9648-406d-a761-39274c1c142c   18.5G     0% /mnt/mount1
xvda                                                                                                                       
├─xvda1                             ext3         1.0   cloudimg-bootfs 95440869-adee-4a95-a781-b8d43a4ae3f8  781.1M    15% /boot
└─xvda2                             ext4         1.0   cloudimg-rootfs d0787949-6e06-438b-af87-88ba54944765   19.8G    10% /
xvdb                                                                                                                       
└─xvdb1                             swap         1     SWAP-xvdb1      d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
xvdh                                vfat         FAT16 config-2        9796-932E    
```

### Show iSCSI Block Volume details from mpath in VSI
{: #show-block-volume-details-from-mpath-cli-example}

1) Get iscsi block device name
```bash
lsblk -f
```

Example output:

```bash
NAME                                FSTYPE       FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                 mpath_member                                                                           
└─3600a098038304750513f4c6374484343                                                                                        
sdb                                 mpath_member                                                                           
└─3600a098038304750513f4c6374484343                                                                                        
xvda                                                                                                                       
├─xvda1                             ext3         1.0   cloudimg-bootfs 95440869-adee-4a95-a781-b8d43a4ae3f8  783.2M    14% /boot
└─xvda2                             ext4         1.0   cloudimg-rootfs d0787949-6e06-438b-af87-88ba54944765     20G    10% /
xvdb                                                                                                                       
└─xvdb1                             swap         1     SWAP-xvdb1      d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
xvdh                                vfat         FAT16 config-2        9796-932E 
```

Iscsi block devices paths in this case are 'sda' and 'sdb'(multipath).

2) Find the target ip of the block volume

Target IP is the ip address that the initiator (VSI) uses to reach the target (iSCSI Block Volume). In order to get the target ip, map the iscsi block device name from previous command to the below command o/p under the 'Attached SCSI devices' and get the Current portal ip which is the target ip of the block volume. In this case '198.51.100.42' is the target ip for the volume attached to iscsi block device path 'sda'.


```bash
iscsiadm -m session -P 3
```

Example output: 

```bash
iSCSI Transport Class version 2.0-870
version 2.1.9
Target: iqn.2004-10.com.ubuntu:01:60f3517884c3 (non-flash)
    Current Portal: 198.51.100.42:3260,1043
    Persistent Portal: 198.51.100.42:3260,1043
        **********
        Interface:
        **********
        Iface Name: default
        Iface Transport: tcp
        Iface Initiatorname: iqn.2004-10.com.ubuntu:01:60f3517884c3
        Iface IPaddress: 10.11.100.154
        Iface HWaddress: default
        Iface Netdev: default
        SID: 1
        iSCSI Connection State: LOGGED IN
        iSCSI Session State: LOGGED_IN
        Internal iscsid Session State: NO CHANGE
        *********
        Timeouts:
        *********
        Recovery Timeout: 5
        Target Reset Timeout: 30
        LUN Reset Timeout: 30
        Abort Timeout: 15
        *****
        CHAP:
        *****
        username: IBM02SU1041833-V153885696
        password: ********
        username_in: <empty>
        password_in: ********
        ************************
        Negotiated iSCSI params:
        ************************
        HeaderDigest: None
        DataDigest: None
        MaxRecvDataSegmentLength: 262144
        MaxXmitDataSegmentLength: 65536
        FirstBurstLength: 65536
        MaxBurstLength: 1048576
        ImmediateData: Yes
        InitialR2T: Yes
        MaxOutstandingR2T: 1
        ************************
        Attached SCSI devices:
        ************************
        Host Number: 2	State: running
        scsi2 Channel 00 Id 0 Lun: 0
            Attached scsi disk sda		State: running
```

If the iSCSI Block volume is multipath enabled , 'multipath' command can be used to check the iSCSI device paths for the block volume.

```bash
$ multipath -ll
67268.950871 | NETAPP/LUN: option 'features "1 queue_if_no_path"' is deprecated, please use 'no_path_retry queue' instead
3600a098038304750513f4c6374484343 dm-0 NETAPP,LUN C-Mode
size=20G features='3 queue_if_no_path pg_init_retries 50' hwhandler='1 alua' wp=rw
|-+- policy='round-robin 0' prio=50 status=active
| `- 2:0:0:0 sda 8:0  active ready running
`-+- policy='round-robin 0' prio=10 status=enabled
  `- 3:0:0:0 sdb 8:16 active ready running

```


 
3) List the iSCSI block volume using target ip


```bash
ibmcloud sl block volume-list | grep -E <target_IP>
```

Example output:

```bash
$ ibmcloud sl block volume-list | grep -E "198.51.100.42"                                            
721685964   IBM02SEL1041833-888         dal13        endurance_block_storage               20            -            -      198.51.100.42    0       0                     1                   -
```

The first column is the id of the block volume . Use ibmcloud sl block volume-detail <block_vol_id> to get more details about the volume.

NOTE: In case of multipath, use grep -E "target_IP1|target_IP2"

Example output:

```bash
ibmcloud sl block volume-list | grep -E "198.51.100.43|198.51.100.44"
722806172   IBM02SEL1041833-891         dal13        endurance_block_storage               20            -            -      198.51.100.43   -       0                     0                   -
```


---


## Discover File Storage
{: #discover-file-storage}

IBM Cloud® File Storage for Classic is persistent, fast, and flexible network-attached, NFS-based File Storage for Classic. In this network-attached storage (NAS) environment, you have total control over your file shares function and performance. File Storage for Classic shares can be connected to up to 64 authorized devices over routed TCP/IP connections for resiliency.

For more information see [Getting started with File Storage for Classic](/docs/FileStorage?topic=FileStorage-getting-started)



### List All File Storage Volumes in the account
{: #list-all-file-storage-volumes-in-the-account-example}

Using CLI -

```bash
ibmcloud sl file volume-list
```

Example output:

```bash
id          username                    datacenter   storage_type                         capacity_gb   bytes_used    IOPs   ip_addr                                 lunId   active_transactions   rep_partner_count   notes
112661718   SL02SEV1041833_63           wdc04        endurance_file_storage               20            0             -      fsf-wdc0401p-fz.service.softlayer.com   -       0                     1                   FAILOVER PI STUCK
112663662   SL02SEV1041833_63_REP_1     dal10        endurance_file_storage_replicant     20            -             -      fsf-dal1002d-fz.adn.networklayer.com    -       0                     0                   Error: replica volume should be deleted first
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Account/{SoftLayer_AccountID}/getNasNetworkStorage'
```

For more information, see [get all NFS File share volumes in the account](https://sldn.softlayer.com/reference/services/SoftLayer_Account/getNasNetworkStorage/)



### Show Full Details for a File Volume
{: #show-full-details-for-file-volume-example}

Using CLI -

```bash
ibmcloud sl file volume-detail <file_vol_id>
```

Example output:

```bash
$ ibmcloud sl file volume-detail 722595986
Name                       Value
ID                         722595986
User name                  IBM02SEV1041833_1179
Type                       endurance_file_storage
Capacity (GB)              20
LUN Id                     -
Endurance Tier             LOW_INTENSITY_TIER
Endurance Tier Per IOPS    0.25
Datacenter                 dal13
Target IP                  fsf-dal1301e-fz.adn.networklayer.com
Mount Address              fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01
# of Active Transactions   0
Replicant Count            0
Notes                      -
Encrypted                  False
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET /
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}?objectMask=mask[id,username,password,capacityGb,bytesUsed,snapshotCapacityGb,parentVolume.snapshotSizeBytes,storageType.keyName,serviceResource.datacenter.name,serviceResourceBackendIpAddress,fileNetworkMountAddress,storageTierLevel,iops,lunId,originalVolumeName,originalSnapshotName,originalVolumeSize,activeTransactionCount,activeTransactions.transactionStatus.friendlyName,replicationPartnerCount,replicationStatus,replicationPartners.id,replicationPartners.username,replicationPartners.serviceResourceBackendIpAddress,replicationPartners.serviceResource.datacenter.name,replicationPartners.replicationSchedule.type.keyname,notes]'
```

For more information, see [get File share volume details](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/)



### Show Snapshot Details for a File Volume
{: #show-snapshot-details-for-file-volume-example}

Using CLI -

```bash
ibmcloud sl file snapshot-list <file_vol_id>
```

Example output:

```bash
ibmcloud sl file snapshot-list 722595986           
id          user_name              created                     size_bytes   notes
722961778   IBM02SEV1041833_1179   2025-10-29T01:06:54-05:00   393216000    IBM02SEV1041833_1179%3A2025-10-29T06%3A06%3A49.677Z
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}/getSnapshots?objectMask=mask[id,username,notes,snapshotSizeBytes,storageType.keyName,snapshotCreationTimestamp,hourlySchedule,dailySchedule,weeklySchedule]'
```

For more information, see [List snapshots for File volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/getSnapshots/)



### Show Replica Details for a File Volume
{: #show-replica-details-for-file-volume-example}

Using CLI -

```bash
ibmcloud sl file replica-partners <file_vol_id>
```

Example output:

```bash
ibmcloud sl file replica-partners 722595986           
ID          User name                    Account ID   Capacity (GB)   Hardware ID   Guest ID   Host ID
722964836   IBM02SEV1041833_1179_REP_1   1041833      20              -             -          -
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}/getReplicationPartners'
```
For more information , see [getReplicationPartners for File volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/getReplicationPartners/)



### Find the authorized VSI details using NFS File Volume details
{: #find-vsi-details-using-file-volume-details-example}

1) Get VSI details

Using CLI -

```bash
ibmcloud sl file access-list <file_vol_id>
```

Example output for VSI that is authorised only for File volume:

```bash
$ ibmcloud sl file access-list 721245700                 
id          name                               type      private_ip_address   source_subnet   host_iqn   username   password   allowed_host_id
154290696   virtualserver01.ibmcloud.private   VIRTUAL   10.01.235.24        -               -                                2904294
```

Example output for VSI that is authorised for both iSCSI and File volume:

```bash                
id          name                               type      private_ip_address   source_subnet   host_iqn                                        username                    password           allowed_host_id
154290696   virtualserver01.ibmcloud.private   VIRTUAL   10.01.235.24        -               iqn.2004-10.com.ubuntu:01:V154290696   IBM02SU1041833-V154290696   ****************   2904294
```

Using API -

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}?objectMask=mask[id,allowedVirtualGuests.id,allowedVirtualGuests.hostname,allowedVirtualGuests.domain,allowedVirtualGuests.primaryBackendIpAddress,allowedVirtualGuests.allowedHost.credential,allowedVirtualGuests.allowedHost.subnetsInAcl,allowedHardware.id,allowedHardware.hostname,allowedHardware.domain,allowedHardware.primaryBackendIpAddress,allowedHardware.allowedHost.credential,allowedHardware.allowedHost.subnetsInAcl,allowedSubnets.id,allowedSubnets.note,allowedSubnets.networkIdentifier,allowedSubnets.cidr,allowedSubnets.endPointIpAddress.ipAddress,allowedSubnets.allowedHost.credential,allowedSubnets.allowedHost.subnetsInAcl,allowedIpAddresses.id,allowedIpAddresses.note,allowedIpAddresses.ipAddress,allowedIpAddresses.allowedHost.credential,allowedIpAddresses.allowedHost.subnetsInAcl]'
```

For more information, see [get authorised VS for the file share volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/)

2) Get mount address
Find the mount address of the NFS file using 'ibmcloud sl file volume-detail <file_vol_id>' command. Refer [View file volume mount address](#show-full-details-for-file-volume-example)

3) Get mount path

Run 'df -h' or 'mount' command in the authorised VSI to find the mount path of the mounted NFS file share. Refer Step1 in [Get name of the File mounted on VSI](#show-file-volume-details-from-mpath-cli-example)



### Show File Volume details from mpath in VSI
{: #show-file-volume-details-from-mpath-cli-example}

1) Get name of the File mounted on VSI.

The mount address contains the name of the file storage volume.

```bash
df -h
```

Example output:

```bash
Filesystem                                                         Size  Used Avail Use% Mounted on
tmpfs                                                              389M  1.1M  388M   1% /run
/dev/xvda2                                                          24G  2.5G   20G  11% /
tmpfs                                                              1.9G     0  1.9G   0% /dev/shm
tmpfs                                                              5.0M     0  5.0M   0% /run/lock
/dev/xvda1                                                         975M  143M  782M  16% /boot
fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01   20G     0   20G   0% /mnt/nfs
tmpfs                                                              389M   12K  389M   1% /run/user/0
```

'IBM02SEV1041833_1179' is the name of the file storage volume in this case. Check the 'Mounted on' column to find the mount path on the VSI.

NOTE: 'mount' command can also be used to find the mount address of the file volume.

```bash
mount
```

Grep for mount point and check for 'type nfs' in the below o/p to find details about mount address of the nfs file share.
Example output:

```bash
fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01 on /mnt/nfs type nfs (rw,relatime,vers=3,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.26.114.44,mountvers=3,mountport=635,mountproto=udp,local_lock=none,addr=192.26.114.44)
```

2) Get File volume details using the name.


Refer [command to get File Volume Detail](#show-full-details-for-file-volume-example)

Example output:

```bash
ibmcloud sl file volume-detail IBM02SEV1041833_1179     
Name                       Value
ID                         722595986
User name                  IBM02SEV1041833_1179
Type                       endurance_file_storage
Capacity (GB)              20
LUN Id                     -
Endurance Tier             LOW_INTENSITY_TIER
Endurance Tier Per IOPS    0.25
Datacenter                 dal13
Target IP                  fsf-dal1301e-fz.adn.networklayer.com
Mount Address              fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01
Snapshot Size (GB)         5
Snapshot Used (Bytes)      159744
# of Active Transactions   0
Replicant Count            1
Replication Status         REPLICATION_PROVISIONING_COMPLETED
Replicant Volumes
                           Replicant ID   722964836
                           Volume Name    IBM02SEV1041833_1179_REP_1
                           Target IP      fsf-dal1401a-fz.adn.networklayer.com
                           Datacenter     dal14
                           Schedule       REPLICATION_HOURLY

Notes                      -
Encrypted                  False
```


### Show NFS usage in VSI
{: #show-nfs-usage-in-vsi-example}

```bash
df -h
```

Example output:

```bash
Filesystem                                                         Size  Used Avail Use% Mounted on
tmpfs                                                              389M  1.1M  388M   1% /run
/dev/xvda2                                                          24G  2.5G   20G  11% /
tmpfs                                                              1.9G     0  1.9G   0% /dev/shm
tmpfs                                                              5.0M     0  5.0M   0% /run/lock
/dev/xvda1                                                         975M  143M  782M  16% /boot
fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01   20G     0   20G   0% /mnt/nfs
tmpfs                                                              389M   12K  389M   1% /run/user/0
```

In this case size of the file storage volume is 20G, '/mnt/nfs' is the mount dir and available storage is 20G.


---


## Discover Local/Portable Storage
{: #discover-local-portable-storage}

Portable storage is an ideal solution if you want to transfer data between virtual servers that exist in any data center on IBM Cloud's network. The portable SAN is built on IBM Cloud's all Flash Storage clusters rather than the local host storage. This infrastructure provides greater resiliency if a host fails and can also support much larger volumes. If a host fails, virtual server instances that use SAN-based storage are automatically migrated to other hosts and restarted. For more information on attaching and managing portable storage, see [Attach Portable Storage](/docs/virtual-servers?topic=virtual-servers-accessing-portable-storage).

Local storage is built on disks that are local to the virtual server host. Local storage provides improved disk read and write performance. The disks are built in a redundant array of independent disks (RAID) configuration for redundancy, disk replacement, and health monitoring that is fully managed by IBM Cloud®. In newer data centers, this storage is all solid-state drive (SSD) to provide the best performance. For more information about available data centers for local SSD storage, see
[Balanced Local Storage](/docs/virtual-servers?topic=virtual-servers-about-virtual-server-profiles#balanced-local-storage)



### List Portable Storage attached to VSI
{: #list-portable-storage-attached-to-vsi-example}

Using CLI -

```bash
ibmcloud sl vs storage <vsi_id>
```
Refer 'Portable Storage' section in the o/p to find Description, Capacity and Location of the Portable Storage attached to VSI

Example output: Refer [Example](#list-all-storage-attached-to-virtual-servers-cli-example)

Using API - 

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Virtual_Guest/{VSI_ID}?objectMask=mask[hostname,id,operatingSystem.softwareLicense.softwareDescription.longDescription,provisionDate,frontendNetworkComponents.speed,frontendNetworkComponents.primaryIpAddress,maxMemory,maxCpu,backendNetworkComponents.speed,backendNetworkComponents.primaryIpAddress,allowedNetworkStorage,blockDevices.diskImage,localDiskFlag,blockDevices.bootableFlag,blockDevices.device,blockDevices.diskImage.capacity,blockDevices.diskImage.diskImageStorageGroup,blockDevices.diskImage.importedDiskType,blockDevices.diskImage.localDiskFlag,blockDevices.diskImage.description]'
```

Example output:

You can find the portable storages under blockDevices -> bootableFlag:0. Refer "description" value to map the respective portable storage volume.

```bash
{"hostname":"virtualserver01","id":154195996,"maxCpu":2,"maxMemory":4096,"provisionDate":"2025-11-06T07:43:52-06:00","allowedNetworkStorage":[{"accountId":1041833,"capacityGb":20,"createDate":"2025-10-30T02:21:22-07:00","guestId":null,"hardwareId":null,"hostId":null,"id":723195182,"nasType":"ISCSI","serviceProviderId":1,"storageTypeId":"7","upgradableFlag":true,"username":"IBM02SEL1041833-909","serviceResourceBackendIpAddress":"192.0.2.10","serviceResourceName":"Storage Type 02 Block Aggregate stbf-dal1303g"},{"accountId":1041833,"capacityGb":20,"createDate":"2025-10-21T07:06:09-07:00","guestId":null,"hardwareId":null,"hostId":null,"id":721245700,"nasType":"NAS","notes":"  Automation Storage Test - SMT - MOUNT \"\"","serviceProviderId":1,"storageTypeId":"13","upgradableFlag":true,"username":"IBM02SEV1041833_935","serviceResourceBackendIpAddress":"fsf-dal1301j-fz.adn.networklayer.com","serviceResourceName":"Storage Type 02 File Aggregate stff-dal1301j"}],"backendNetworkComponents":[{"speed":100,"primaryIpAddress":"192.0.2.10"}],"blockDevices":[{"bootableFlag":1,"device":"0","diskImage":{"capacity":100,"description":"virtualserver01.ibmcloud.private","localDiskFlag":true}},{"bootableFlag":0,"device":"1","diskImage":{"capacity":2,"description":"154195996-SWAP","localDiskFlag":true}},{"bootableFlag":0,"device":"2","diskImage":{"capacity":100,"description":"virtualserver01 - Disk 2","localDiskFlag":true}},{"bootableFlag":0,"device":"4","diskImage":{"capacity":10,"description":"virtualserver01 - Disk 3","localDiskFlag":true}},{"bootableFlag":0,"device":"7","diskImage":{"capacity":64,"description":"virtualserver01 - Metadata","localDiskFlag":true}}],"frontendNetworkComponents":[{"speed":100,"primaryIpAddress":"192.0.2.11"}],"localDiskFlag":true,"operatingSystem":{"hardwareId":null,"id":91776354,"manufacturerLicenseInstance":"","softwareLicense":{"id":80628,"softwareDescriptionId":3228,"softwareDescription":{"longDescription":"Ubuntu 24.04-64 Minimal for VSI"}}}}%   
```



### Show Local/Portable Volume details from mpath in VSI
{: #show-local-portable-volume-details-from-mpath-cli-example}

1) Get Mpath details of the file system mounted on VSI

Let us consider the file system mounted on '/mnt/portableStorage' . 

```bash
lsblk -f
```


Example output:

```bash
NAME    FSTYPE FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
xvda                                                                                     
├─xvda1 ext3   1.0   cloudimg-bootfs 95440869-adee-4a95-a781-b8d43a4ae3f8  849.7M     8% /boot
└─xvda2 ext4   1.0   cloudimg-rootfs d0787949-6e06-438b-af87-88ba54944765   91.3G     2% /
xvdb                                                                                     
└─xvdb1 swap   1     SWAP-xvdb1      d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
xvdc                                                                                     
xvde    ext4   1.0                   78c10f59-5a11-4235-ab23-3fcc016efaf5    9.2G     0% /mnt/portableStorage
xvdh    vfat   FAT16 config-2        9796-932E 
```

Refer column 'NAME' to get the block device name on which the file system exist. In this case, 'xvde' contains file stystem that is mounted on '/mnt/portableStorage'.

2) Get the block device number.

Block device numbers start at 0 and correspond to disks in alphabetical order. For example, xvda is device number 0, xvdb is 1, xvdc is 2, and so on.

In this case, xvde is mapped to device number 4.

3) Map the device number to description.

Get the VSI_ID to which the portable storage volume is attached and run below  api call to get the description of block devices attached to the VSI

```bash
curl -g -u $SL_USER:$SL_APIKEY -X GET 'https://api.softlayer.com/rest/v3.1/SoftLayer_Virtual_Guest/{VSI_ID}.json?objectMask=mask[hostname,id,operatingSystem.softwareLicense.softwareDescription.longDescription,provisionDate,frontendNetworkComponents.speed,frontendNetworkComponents.primaryIpAddress,maxMemory,maxCpu,backendNetworkComponents.speed,backendNetworkComponents.primaryIpAddress,allowedNetworkStorage,blockDevices.diskImage,localDiskFlag,blockDevices.bootableFlag,blockDevices.device,blockDevices.diskImage.capacity,blockDevices.diskImage.diskImageStorageGroup,blockDevices.diskImage.importedDiskType,blockDevices.diskImage.localDiskFlag,blockDevices.diskImage.description]'
```

Under 'blockDevices' you will find a list of all the block devices attached. Map the 'device' field value to the number from previous step and see 'diskImage' field to get information on 'description' and 'capacity' of the portable storage volume.


Example output:

```bash
{
  "hostname": "virtualserver01",
  "id": 154195996,
  "maxCpu": 2,
  "maxMemory": 4096,
  "provisionDate": "2025-11-06T07:43:52-06:00",
  "localDiskFlag": true,
  "operatingSystem": 
  {
    "id": 91776354,
    "manufacturerLicenseInstance": "",
    "softwareLicense": 
    {
      "id": 80628,
      "softwareDescriptionId": 3228,
      "softwareDescription": 
      {
        "longDescription": "Ubuntu 24.04-64 Minimal for VSI"
      }
    }
  },
  "frontendNetworkComponents": [
    {
      "speed": 100,
      "primaryIpAddress": "192.0.2.11"
    }
  ],
  "backendNetworkComponents": [
    {
      "speed": 100,
      "primaryIpAddress": "192.0.2.10"
    }
  ],
  "blockDevices": [
    {
      "bootableFlag": 1,
      "device": "0",
      "diskImage": 
      {
        "capacity": 100,
        "description": "virtualserver01.ibmcloud.private",
        "localDiskFlag": true
      }
    },
    {
      "bootableFlag": 0,
      "device": "1",
      "diskImage": 
      {
        "capacity": 2,
        "description": "154195996-SWAP",
        "localDiskFlag": true
      }
    },
    {
      "bootableFlag": 0,
      "device": "2",
      "diskImage": 
      {
        "capacity": 100,
        "description": "virtualserver01 - Disk 2",
        "localDiskFlag": true
      }
    },
    {
      "bootableFlag": 0,
      "device": "4",
      "diskImage": 
      {
        "capacity": 10,
        "description": "virtualserver01 - Disk 3",
        "localDiskFlag": true
      }
    },
    {
      "bootableFlag": 0,
      "device": "7",
      "diskImage": 
      {
        "capacity": 64,
        "description": "virtualserver01 - Metadata",
        "localDiskFlag": true
      }
    }
  ],
  "allowedNetworkStorage": [
    {
      "accountId": 1041833,
      "capacityGb": 20,
      "createDate": "2025-10-30T02:21:22-07:00",
      "id": 723195182,
      "nasType": "ISCSI",
      "serviceProviderId": 1,
      "storageTypeId": "7",
      "upgradableFlag": true,
      "username": "IBM02SEL1041833-909",
      "serviceResourceBackendIpAddress": "192.0.2.12",
      "serviceResourceName": "Storage Type 02 Block Aggregate stbf-dal1303g"
    },
    {
      "accountId": 1041833,
      "capacityGb": 20,
      "createDate": "2025-10-21T07:06:09-07:00",
      "id": 721245700,
      "nasType": "NAS",
      "notes": "Automation Storage Test - SMT - MOUNT \"\"",
      "serviceProviderId": 1,
      "storageTypeId": "13",
      "upgradableFlag": true,
      "username": "IBM02SEV1041833_935",
      "serviceResourceBackendIpAddress": "fsf-dal1301j-fz.adn.networklayer.com",
      "serviceResourceName": "Storage Type 02 File Aggregate stff-dal1301j"
    }
  ]
}
```

In this case, the device number is 4. Therefore the portable storage volume that is mapped to /mnt/portableStorage Mount point is 'virtualserver01 - Disk 3' with capacity of 10 GB.



### Find mount point in VSI for a particular Local/Portable Storage Volume
{: #find-mount-point-from-portable-volume-details-cli-example}

1) Find device number of the local/portable storage volume attached

The 'diskImage' field under 'blockDevices' contains details of the portable storage volume including device number. Refer Step 3 [Get device number](#show-local-portable-volume-details-from-mpath-cli-example).

2) Map the device number to block device name.

Block device numbers start at 0 and correspond to disks in alphabetical order. For example, xvda is device number 0, xvdb is 1, xvdc is 2, and so on.

3) Map the block device names to mount point

In below case , file system exists on xvde (Refer NAME column) for which the mount point is /mnt/portableStorage (Refer MOUNTPOINTS column) . 

Example output:

```bash
$ lsblk -f
NAME    FSTYPE FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
xvda                                                                                     
├─xvda1 ext3   1.0   cloudimg-bootfs 95440869-adee-4a95-a781-b8d43a4ae3f8  849.7M     8% /boot
└─xvda2 ext4   1.0   cloudimg-rootfs d0787949-6e06-438b-af87-88ba54944765   91.3G     2% /
xvdb                                                                                     
└─xvdb1 swap   1     SWAP-xvdb1      d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
xvdc                                                                                     
xvde    ext4   1.0                   78c10f59-5a11-4235-ab23-3fcc016efaf5    9.2G     0% /mnt/portableStorage
xvdh    vfat   FAT16 config-2        9796-932E 
```
