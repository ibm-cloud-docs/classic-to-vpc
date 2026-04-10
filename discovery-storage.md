---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: storage, classic virtual server instance

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Discovery of classic storage resources
{: #discover-classic-storage-resources}

{{site.data.keyword.cloud}} classic infrastructure offers the following storage options:

- **Local storage** is built on disks that are local to the virtual server host. Local storage provides improved disk read and write performance. For more information about available data centers for local SSD storage, see [Balanced local storage](/docs/virtual-servers?topic=virtual-servers-about-virtual-server-profiles#balanced-local-storage).

- **Block storage** is persistent, high-performance iSCSI storage that is provisioned and managed independently of compute instances. For more information about provisioning, see [Block storage provisioning](/docs/BlockStorage?group=ordering-storage).

- **File storage** is persistent, fast, and flexible network-attached, NFS-based file storage for Classic. For more information about provisioning file share, see [File storage provisioning](/docs/FileStorage?group=ordering-storage).

- **Portable SAN storage** volumes are auxiliary storage solutions that are available exclusively on virtual servers. For more information about attaching portable storage, see [attach portable storage](/docs/virtual-servers?topic=virtual-servers-accessing-portable-storage#attaching-portable-storage)

## Information to collect
{: #storage-information-to-collect}

When you prepare for migration, gather the following information for each classic storage type:

- Storage type: block storage, file storage, portable storage, and local storage.
- Profile type: Endurance or Performance type for block and file.
- Volume IOPS: IOPS of the storage volume that is attached to the device.
- Volume capacity: Size of the storage volume that is attached to the device.
- Capacity usage: Size of the space on the block or file storage volume that is used in the device.
- Snapshots: Snapshot details for block and file storage.
- Replica Volume: Replica details for block and file storage.

You can query classic infrastructure resources with the `ibmcloud sl` CLI plug-in.

### Listing all storage volumes that are attached to virtual servers from the CLI
{: #list-all-storage-attached-to-virtual-servers-cli-example}
{: cli}

```sh
ibmcloud sl vs storage VSI_ID
```
{: pre}

Example output:

```sh
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
{: screen}

### Listing all storage volumes that are attached to bare metal Servers from the CLI
{: #list-all-storage-attached-to-bare-metal-cli-example}

```sh
ibmcloud sl hardware storage BM_ID
```
{: pre}

Example output:

```sh
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
{: screen}

## Discovering block storage
{: #discover-block-storage}

Block storage brings best-in-class durability and availability with an unmatched feature set. It is built with the help of industry standards and best practices. Block storage is designed to protect data integrity, maintain availability during maintenance events and unplanned failures, and provide a consistent performance baseline.

For more information, see [Getting started with block storage for Classic](/docs/BlockStorage?topic=BlockStorage-getting-started)

### Listing all iSCSI block storage volumes in the account
{: #list-all-iscsi-block-storage-volumes-in-the-account-example}

#### From the CLI
{: #list-all-block-volumes-cli}
{: cli}

```sh
ibmcloud sl block volume-list
```
{: pre}

Example output:

```sh
id          username                    datacenter   storage_type                          capacity_gb   bytes_used   IOPs   ip_addr          lunId   active_transactions   rep_partner_count   notes
195144982   SL02SL1041833_63_REP_1      -            performance_block_storage_replicant   20            -            100    192.0.2.10     -       0                     0                   -
217954031   SL02SEL1041833-537          dal10        endurance_block_storage               35            -            -      192.0.2.11    25      0                     0                   DISASTER RECOVERY FAILOVER COMPLETED
227445958   SL02SEL1041833-512          dal10        endurance_block_storage               100           -            -      192.0.2.12    7       0                     0                   DISASTER RECOVERY FAILOVER COMPLETED + FAILBACK COMPLETED
```
{: screen}

#### With the API
{: #list-all-block-volumes-api}
{: api}

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Account/{SoftLayer_AccountID}/getIscsiNetworkStorage'
```
{: pre}

For more information, see [List all block volumes in the account](https://sldn.softlayer.com/reference/services/SoftLayer_Account/getIscsiNetworkStorage/){: external}

### Show full details for an iSCSI block volume example
{: #show-full-details-for-iscsi-block-volume-example}

#### From the CLI
{: #show-details-block-volumes-cli}
{: cli}

```sh
ibmcloud sl block volume-detail VOLUME_ID
```
{: pre}

Example output:

```sh
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
{: screen}

#### With the API
{: #show-details-block-volumes-api}
{: api}

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{Block_Vol_ID}?objectMask=mask[id,username,password,capacityGb,snapshotCapacityGb,parentVolume.snapshotSizeBytes,storageType.keyName,serviceResource.datacenter.name,serviceResourceBackendIpAddress,storageTierLevel,iops,lunId,originalVolumeName,originalSnapshotName,originalVolumeSize,hasEncryptionAtRest,activeTransactionCount,activeTransactions.transactionStatus.friendlyName,replicationPartnerCount,replicationStatus,replicationPartners.id,replicationPartners.username,replicationPartners.serviceResourceBackendIpAddress,replicationPartners.serviceResource.datacenter.name,replicationPartners.replicationSchedule.type.keyname,notes]'
```
{: pre}

For more information, see [get block volume details](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/){: external}.

### Showing snapshot details for an iSCSI block volume
{: #show-snapshot-details-for-iscsi-block-volume-example}

#### From the CLI
{: #show-snapshot-details-block-volumes-cli}
{: cli}

```sh
ibmcloud sl block snapshot-list VOLUME_ID
```
{: pre}

Example output:

```sh
ibmcloud sl block snapshot-list 721685964
id          user_name              created                     size_bytes   notes
722754994   IBM02SEVC1041833_943   2025-10-28T01:20:00-05:00   163840       -
```
{: screen}

#### With the API
{: #show-snapshot-details-block-volumes-api}
{: api}

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{Block_Vol_ID}/getSnapshots?objectMask=mask[id,username,notes,snapshotSizeBytes,storageType.keyName,snapshotCreationTimestamp,hourlySchedule,dailySchedule,weeklySchedule]'
```
{: pre}

For more information, see [List snapshots for iSCSI block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getSnapshots/){: external}

### Showing replica details for an iSCSI block volume
{: #show-replica-details-block-volume}

#### From the CLI
{: #show-replica-details-block-volumes-cli}
{: cli}

```sh
ibmcloud sl block replica-partners VOLUME_ID
```
{: pre}

Example output:

```sh
ibmcloud sl block replica-partners 721685964
ID          User name                   Account ID   Capacity (GB)   Hardware ID   Guest ID   Host ID
721870496   IBM02SEL1041833_888_REP_1   1041833      20              -             -          -
```
{: screen}

### With the API
{: #show-replica-details-block-volumes-api}
{: api}

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{SoftLayer_Network_Storage_IscsiID}/getReplicationPartners'
```
{: pre}

For more information, see [getReplicationPartners for iSCSI block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getReplicationPartners/){: external}

### Find the mount path in a virtual server instance by using iSCSI block volume details example
{: #find-mount-path-in-vsi-using-iscsi-block-volume-details-example}

1. Get the virtual server instance details that are authorized to use an iSCSI block volume.

   From the CLI:
   {: cli}

   ```sh
   ibmcloud sl block access-list VOLUME_ID
   ```
   {: pre}
   {: cli}

   Example output:
   {: cli}

   ```sh
   $ ibmcloud sl block access-list 721685964

   id          name                                  type      private_ip_address   source_subnet   host_iqn                                        username                    password     allowed_host_id
   153885696   poc-classic-to-vpc.ibmcloud.private   VIRTUAL   203.0.113.99         -               iqn.2004-10.com.ubuntu:01:V153885696            IBM02SU1041833-V153885696   **********   2872634
   ```
   {: screen}
   {: cli}

   Run `ibmcloud sl vs detail VSI_ID` to get more details on the virtual server instance. You can find the virtual server instance ID in the first column in the output. In this case, the virtual server instance ID is `153885696`.
   {: cli}

   With the API:
   {: api}

   ```sh
   curl -g -u $SL_USER:$SL_APIKEY -X GET \
   'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage_Iscsi/{SoftLayer_Network_Storage_IscsiID}/getAllowedVirtualGuests'
   ```
   {: pre}
   {: api}

   For more information, see [get all authorized VS for a particular iSCSI block volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage_Iscsi/getAllowedVirtualGuests/){: external}.
   {: api}

   Run the following API call to get more details on a particular virtual server instance. You can find the virtual server instance ID in the output of the previous API call.
   {: api}

   ```sh
   curl -g -u $SL_USER:$SL_APIKEY -X GET \
   'https://api.softlayer.com/rest/v3.1/SoftLayer_Virtual_Guest/{VSI_ID}'
   ```
   {: pre}
   {: api}

   For more information, see [get virtual server instance details](https://sldn.softlayer.com/reference/services/SoftLayer_Virtual_Guest/){: external}.
   {: api}

2. Get the target IP address of the iSCSI block volume. Refer to Step 2 in [Finding the target IP address of the block volume](#show-full-details-for-an-iscsi-block-volume-example).

3. Get the iSCSI block device path.

    Run `iscsiadm -m session -P 3` command within the virtual server instance. Match the target IP address that is obtained from Step 2 to the current portal IP address in the output. Then, under **Attached SCSI devices**, locate the **Attached scsi disk** to identify the iSCSI block device path on which the block volume is attached.

   Example output:

    ```sh
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
        Host Number: 2State: running
        scsi2 Channel 00 Id 0 Lun: 0
            Attached scsi disk sdaState: running

    ```
    {: screen}

4. Get mount path details. Run `lsblk -f` command to get the mount path, file system type, UUID, and available space of the file system mounted on the multipath device. In this example, sda and sdb represent iSCSI device paths on which the ext4 file system is mounted on path - /mnt/mount1. Use the file system mount path for data migration to {{site.data.keyword.vpc_full}}.

  Example output:

  ```sh
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
    ├─xvda1                             ext3         1.0   cloudimg-bootfs   95440869-adee-4a95-a781-b8d43a4ae3f8  781.1M    15% /boot
    └─xvda2                             ext4         1.0   cloudimg-rootfs d0787949-6e06-438b-af87-88ba54944765   19.8G    10% /
    xvdb
    └─xvdb1                             swap         1     SWAP-xvdb1      d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
    xvdh                                vfat         FAT16 config-2        9796-932E
  ```
  {: screen}

### Showing iSCSI block volume details from mpath in a virtual server instance example
{: #show-block-volume-details-from-mpath-cli-example}

1. Get the iSCSI block device name.

    ```sh
    sblk -f
    ```
    {: pre}

    Example output:

    ```sh
    NAME                                FSTYPE       FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
    sda                                 mpath_member
    └─3600a098038304750513f4c6374484343
    sdb                                 mpath_member
    └─3600a098038304750513f4c6374484343
    xvda
    ├─xvda1                             ext3         1.0   cloudimg-bootfs   95440869-adee-4a95-a781-b8d43a4ae3f8  783.2M    14% /boot
    └─xvda2                             ext4         1.0   cloudimg-rootfs   d0787949-6e06-438b-af87-88ba54944765     20G    10% /
    xvdb
    └─xvdb1                             swap         1     SWAP-xvdb1        d51fcca0-6b10-4934-a572-f3898dfd8840                [SWAP]
    xvdh                                vfat         FAT16 config-2        9796-932E
    ```
    {: screen}

    In this example, the iSCSI block device's paths are sda and sdb (multipath).

2. Find the target IP address of the block volume

    The target IP address is the address that the initiator virtual server instance uses to reach the target (iSCSI block Volume). To find the target IP address, map the iSCSI block device name from previous command to the following command output under **Attached SCSI devices** and get the `Current portal IP`, which is the target IP address of the block volume. In this example, `198.51.100.42` is the target IP address for the volume that is attached to iSCSI block device path sda.

    ```sh
    iscsiadm -m session -P 3
    ```
    {: pre}

    Example output:

      ```sh
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
              Host Number: 2State: running
              scsi2 Channel 00 Id 0 Lun: 0
                  Attached scsi disk sdaState: running
      ```
      {: screen}

    If the iSCSI block volume is multipath that is enabled, use the 'multipath' command to check the iSCSI device paths for the block volume.

    ```sh
     multipath -ll
          67268.950871 | NETAPP/LUN: option 'features "1 queue_if_no_path"' is deprecated, please use 'no_path_retry queue' instead
          3600a098038304750513f4c6374484343 dm-0 NETAPP,LUN C-Mode
        size=20G features='3 queue_if_no_path pg_init_retries 50' hwhandler='1 alua' wp=rw
          |-+- policy='round-robin 0' prio=50 status=active
          | `- 2:0:0:0 sda 8:0  active ready running
          `-+- policy='round-robin 0' prio=10 status=enabled
            `- 3:0:0:0 sdb 8:16 active ready running
      ```
    {: screen}

3. Use the target IP address to list the iSCSI block volume

    ```sh
    ibmcloud sl block volume-list | grep -E TARGET_IP
    ```
    {: pre}

    Example output:

    ```sh
    $ ibmcloud sl block volume-list | grep -E "198.51.100.42"
    721685964   IBM02SEL1041833-888         dal13        endurance_block_storage               20            -            -      198.51.100.42    0       0                     1                   -
    ```
    {: screen}

    The first column shows the block volume ID. To view more details, use `ibmcloud sl block volume-detail VOLUME_ID`.

    In a multipath configuration, use `grep -E "TARGET_IP1|TARGET_IP2"`.
    {: note}

    Example output:

    ```sh
    ibmcloud sl block volume-list | grep -E "198.51.100.43|198.51.100.44"
    722806172   IBM02SEL1041833-891         dal13        endurance_block_storage               20            -            -      198.51.100.43   -       0                     0                   -
    ```
    {: screen}

### Showing ISCSI usage in a virtual server instance
{: #show-iscsi-usage-in-vsi-example}

From the command line of the virtual server, run one of these commands: `df -k` or `lsblk`. In both examples, `/dev/mapper/3600a098038313871673f56754a724c34` is the block device.

```sh
# df -k
Filesystem                                              1K-blocks      Used Available Use% Mounted on
devtmpfs                                                     4096         0      4096   0% /dev
tmpfs                                                      863972         0    863972   0% /dev/shm
tmpfs                                                      345592     32548    313044  10% /run
/dev/xvda2                                              102111832   3773432  93131652   4% /
/dev/xvda1                                                 996780    365300    562668  40% /boot
tmpfs                                                      172792         4    172788   1% /run/user/1000
nfssjc0301a-fz.service.softlayer.com:/DSW02SEV3134915_1 104857600      3712 104853888   1% /mnt/DSW02SEV3134915_1
/dev/mapper/3600a098038313871673f56754a724c34           514937088 286463152 202243152  59% /mnt/DSW02SEL3134915-2
tmpfs                                                      172792         4    172788   1% /run/user/0
```
{: screen}

```sh
[root@storage-poc-sjc ~]# lsblk
NAME                                MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                                   8:0    0  500G  0 disk
└─3600a098038313871673f56754a724c34 253:0    0  500G  0 mpath /mnt/DSW02SEL3134915-2
sdb                                   8:16   0  500G  0 disk
└─3600a098038313871673f56754a724c34 253:0    0  500G  0 mpath /mnt/DSW02SEL3134915-2
xvda                                202:0    0  100G  0 disk
├─xvda1                             202:1    0    1G  0 part  /boot
└─xvda2                             202:2    0   99G  0 part  /
xvdb                                202:16   0    2G  0 disk
└─xvdb1                             202:17   0    2G  0 part  [SWAP]
xvdc                                202:32   0  100G  0 disk
xvde                                202:64   0  100G  0 disk
xvdh                                202:112  0   64M  0 disk
```
{: screen}

The `/dev/mapper/3600a098038313871673f56754a724c34` volume has 500 GiB capacity, and 59% of the capacity is in use.

```sh
[root@storage-poc-sjc ~]# df -k
Filesystem                                              1K-blocks      Used Available Use% Mounted on
devtmpfs                                                     4096         0      4096   0% /dev
tmpfs                                                      863972         0    863972   0% /dev/shm
tmpfs                                                      345592     32548    313044  10% /run
/dev/xvda2                                              102111832   3773432  93131652   4% /
/dev/xvda1                                                 996780    365300    562668  40% /boot
tmpfs                                                      172792         4    172788   1% /run/user/1000
nfssjc0301a-fz.service.softlayer.com:/DSW02SEV3134915_1 104857600      3712 104853888   1% /mnt/DSW02SEV3134915_1
/dev/mapper/3600a098038313871673f56754a724c34           514937088 286463152 202243152  59% /mnt/DSW02SEL3134915-2
tmpfs                                                      172792         4    172788   1% /run/user/0
```
{: screen}

In this example, `/dev/mapper/3600a098038313871673f56754a724c34` is the block device.

Example output:

```sh
[root@storage-poc-sjc ~]# lsblk
NAME                                MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                                   8:0    0  500G  0 disk
└─3600a098038313871673f56754a724c34 253:0    0  500G  0 mpath /mnt/DSW02SEL3134915-2
sdb                                   8:16   0  500G  0 disk
└─3600a098038313871673f56754a724c34 253:0    0  500G  0 mpath /mnt/DSW02SEL3134915-2
xvda                                202:0    0  100G  0 disk
├─xvda1                             202:1    0    1G  0 part  /boot
└─xvda2                             202:2    0   99G  0 part  /
xvdb                                202:16   0    2G  0 disk
└─xvdb1                             202:17   0    2G  0 part  [SWAP]
xvdc                                202:32   0  100G  0 disk
xvde                                202:64   0  100G  0 disk
xvdh                                202:112  0   64M  0 disk
```
{: screen}

## Discovering file storage
{: #discover-file-storage}

{{site.data.keyword.cloud_notm}} file storage for classic is persistent, fast, and flexible network-attached storage based on NFS. In this network-attached storage (NAS) environment, you have full control over the functions and performance of your file shares. These shares can connect to up to 64 authorized devices over routed TCP/IP address connections for improved resiliency.

For more information, see [Getting started with file storage for Classic](/docs/FileStorage?topic=FileStorage-getting-started)

### Listing all file storage volumes in the account
{: #list-all-file-storage-volumes-in-the-account-example}

From the CLI, run the following command.

```sh
ibmcloud sl file volume-list
``` {: pre}

Example output:

```sh
id          username                    datacenter   storage_type                         capacity_gb   bytes_used    IOPs   ip_addr                                 lunId   active_transactions   rep_partner_count   notes
112661718   SL02SEV1041833_63           wdc04        endurance_file_storage               20            0             -      fsf-wdc0401p-fz.service.softlayer.com   -       0                     1                   FAILOVER PI STUCK
112663662   SL02SEV1041833_63_REP_1     dal10        endurance_file_storage_replicant     20            -             -      fsf-dal1002d-fz.adn.networklayer.com    -       0                     0                   Error: replica volume should be deleted first
```
{: screen}

With the API -

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Account/{SoftLayer_AccountID}/getNasNetworkStorage'
```
{: pre}

For more information, see [get all NFS File share volumes in the account](https://sldn.softlayer.com/reference/services/SoftLayer_Account/getNasNetworkStorage/){: external}

### Showing full details for file volume
{: #show-full-details-for-file-volume-example}

From the CLI, run the following command.

```sh
ibmcloud sl file volume-detail VOLUME_ID
```
{: pre}

Example output:

```sh
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
{: screen}

With the API -

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET /
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}?objectMask=mask[id,username,password,capacityGb,bytesUsed,snapshotCapacityGb,parentVolume.snapshotSizeBytes,storageType.keyName,serviceResource.datacenter.name,serviceResourceBackendIpAddress,fileNetworkMountAddress,storageTierLevel,iops,lunId,originalVolumeName,originalSnapshotName,originalVolumeSize,activeTransactionCount,activeTransactions.transactionStatus.friendlyName,replicationPartnerCount,replicationStatus,replicationPartners.id,replicationPartners.username,replicationPartners.serviceResourceBackendIpAddress,replicationPartners.serviceResource.datacenter.name,replicationPartners.replicationSchedule.type.keyname,notes]'
```
{: pre}

For more information, see [get File share volume details](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/){: external}

### Showing snapshot details for a file volume
{: #show-snapshot-details-for-file-volume-example}

From the CLI, run the following command.

```sh
ibmcloud sl file snapshot-list VOLUME_ID
```
{: pre}

Example output:

```sh
ibmcloud sl file snapshot-list 722595986
id          user_name              created                     size_bytes   notes
722961778   IBM02SEV1041833_1179   2025-10-29T01:06:54-05:00   393216000    IBM02SEV1041833_1179%3A2025-10-29T06%3A06%3A49.677Z
```
{: screen}

With the API -

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}/getSnapshots?objectMask=mask[id,username,notes,snapshotSizeBytes,storageType.keyName,snapshotCreationTimestamp,hourlySchedule,dailySchedule,weeklySchedule]'
```
{: pre}

For more information, see [List snapshots for file volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/getSnapshots/){: external}

### Showing replica details for a file volume
{: #show-replica-details-for-file-volume-example}

From the CLI, run the following command.

```sh
ibmcloud sl file replica-partners VOLUME_ID
```
{: pre}

Example output:

```sh
ibmcloud sl file replica-partners 722595986
ID          User name                    Account ID   Capacity (GB)   Hardware ID   Guest ID   Host ID
722964836   IBM02SEV1041833_1179_REP_1   1041833      20              -             -          -
```
{: screen}

With the API -

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}/getReplicationPartners'
```
{: pre}

For more information, see [getReplicationPartners for file volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/getReplicationPartners/){: external}

### Finding the authorized virtual server instance details by using file volume details
{: #find-vsi-details-using-file-volume-details-example}

1. Get the virtual server instance details

    From the CLI, run the following command.

    ```sh
    ibmcloud sl file access-list FILE_VOLUME_ID
    ```
    {: pre}

    Example output for virtual server instance authorized only for file volume:

    ```sh
     $ ibmcloud sl file access-list 721245700
    id          name                               type      private_ip_address   source_subnet   host_iqn   username   password   allowed_host_id
    154290696   virtualserver01.ibmcloud.private   VIRTUAL   10.01.235.24        -               -                                2904294
    ```
    {: screen}

    Example output for virtual server instance that is authorized for both block and file volumes:

    ```sh
    id          name                               type      private_ip_address   source_subnet   host_iqn                                        username                    password           allowed_host_id
    154290696   virtualserver01.ibmcloud.private   VIRTUAL   10.01.235.24        -               iqn.2004-10.com.ubuntu:01:V154290696   IBM02SU1041833-V154290696   ****************   2904294
     ```
    {: screen}

    With the API -

    ```sh
    curl -g -u $SL_USER:$SL_APIKEY -X GET \
    'https://api.softlayer.com/rest/v3.1/SoftLayer_Network_Storage/{File_Vol_ID}?objectMask=mask[id,allowedVirtualGuests.id,allowedVirtualGuests.hostname,allowedVirtualGuests.domain,allowedVirtualGuests.primaryBackendIpAddress,allowedVirtualGuests.allowedHost.credential,allowedVirtualGuests.allowedHost.subnetsInAcl,allowedHardware.id,allowedHardware.hostname,allowedHardware.domain,allowedHardware.primaryBackendIpAddress,allowedHardware.allowedHost.credential,allowedHardware.allowedHost.subnetsInAcl,allowedSubnets.id,allowedSubnets.note,allowedSubnets.networkIdentifier,allowedSubnets.cidr,allowedSubnets.endPointIpAddress.ipAddress,allowedSubnets.allowedHost.credential,allowedSubnets.allowedHost.subnetsInAcl,allowedIpAddresses.id,allowedIpAddresses.note,allowedIpAddresses.ipAddress,allowedIpAddresses.allowedHost.credential,allowedIpAddresses.allowedHost.subnetsInAcl]'
    ```
    {: pre}

    For more information, see [get authorized VS for the file share volume](https://sldn.softlayer.com/reference/services/SoftLayer_Network_Storage/){: external}

2. Get mount address: use the `ibmcloud sl file volume-detail FILE_VOLUME_ID` command to find the mount address of the NFS file. Refer to [View file volume mount address](#showing-full-details-for-file-volume)

3. Get mount path: run the `df -h` or `mount` command in the authorized virtual server instance to find the NFS file share mount path. Refer to Step1 in [Get the name of the File mounted on a virtual server instance](#showing-file-volume-details-from-mpath-cli)

### Showing file volume details from mpath CLI
{: #show-file-volume-details-from-mpath-cli-example}

1. Get the name of the file mounted on the virtual server instance. The mount address includes the file storage volume name.

   ```sh
   df -h
   ```
   {: pre}

   Example output:

   ```sh
    Filesystem                                                         Size  Used Avail Use% Mounted on
    tmpfs                                                              389M  1.1M  388M   1% /run
    /dev/xvda2                                                          24G  2.5G   20G  11% /
    tmpfs                                                              1.9G     0  1.9G   0% /dev/shm
    tmpfs                                                              5.0M     0  5.0M   0% /run/lock
    /dev/xvda1                                                         975M  143M  782M  16% /boot
    fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01   20G     0   20G   0% /mnt/nfs
    tmpfs                                                              389M   12K  389M   1% /run/user/0
   ```
   {: screen}

    In this example, `IBM02SEV1041833_1179` is the file storage volume name. Check the `Mounted on` column to find the mount path on the virtual server instance.

    Use `mount` command to find the mount address of the file volume.
    {: note}

     ```sh
     mount
     ```
     {: pre}

    Grep for `mount point` and check for `type nfs` in the following output to find details about the mount address of the nfs file share.

    Example output:

    ```sh
    fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01 on /mnt/nfs type nfs (rw,relatime,vers=3,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.26.114.44,mountvers=3,mountport=635,mountproto=udp, local_lock=none,addr=192.26.114.44)
    ```
    {: pre}

2. Get the file volume details by using the name. For more information, see the [command to get file volume detail](#showing-full-details-for-file-volume).

    Example output:

    ```sh
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
    {: screen}

### Showing NFS usage in a virtual server instance
{: #show-nfs-usage-in-vsi-example}

From the command line of the virtual server, run the following command.

```sh
df -h
```
{: pre}

Example output:

```sh
Filesystem                                                         Size  Used Avail Use% Mounted on
tmpfs                                                              389M  1.1M  388M   1% /run
/dev/xvda2                                                          24G  2.5G   20G  11% /
tmpfs                                                              1.9G     0  1.9G   0% /dev/shm
tmpfs                                                              5.0M     0  5.0M   0% /run/lock
/dev/xvda1                                                         975M  143M  782M  16% /boot
fsf-dal1301e-fz.adn.networklayer.com:/IBM02SEV1041833_1179/data01   20G     0   20G   0% /mnt/nfs
tmpfs                                                              389M   12K  389M   1% /run/user/0
```
{: screen}

In this example, the size of the file storage volume is 20G. The mount directory is `/mnt/nfs`, and the available storage is 20G.

## Discovering local and portable storage
{: #discover-local-portable-storage}

Portable storage is an ideal solution if you want to transfer data between virtual servers that exist in any data center on the {{site.data.keyword.cloud_notm}} network. The portable SAN is built on {{site.data.keyword.cloud_notm}} all-flash storage clusters rather than the local host storage. If a host fails, this infrastructure provides greater resiliency and can also support much larger volumes. Also, if a host fails, virtual server instances that use SAN-based storage are automatically migrated to other hosts and restarted. For more information on attaching and managing portable storage, see [Attach Portable Storage](/docs/virtual-servers?topic=virtual-servers-accessing-portable-storage).

Local storage is built on disks that are local to the virtual server host. Local storage provides improved disk read and write performance. The disks are configured in a redundant array of independent disks (RAID) configuration for redundancy, disk replacement, and health monitoring that is fully managed by {{site.data.keyword.cloud_notm}}. In newer data centers, this storage uses all solid-state drives (SSD) to provide the best performance. For more information about available data centers for local SSD storage, see
[Balanced Local Storage](/docs/virtual-servers?topic=virtual-servers-about-virtual-server-profiles#balanced-local-storage)

### Listing portable storage that is attached to a virtual server instance
{: #list-portable-storage-attached-to-vsi-example}

From the CLI, run the following command.

```sh
ibmcloud sl vs storage VSI_ID
```
{: pre}

Refer to the **Portable Storage** section in the output to find its **description**, **capacity**, and **location** of the portable storage attached to the virtual server instance.

Example output: Refer to [Example](#listing-all-storage-volumes-attached-to-virtual-servers-from-the-cli)

With the API -

```sh
curl -g -u $SL_USER:$SL_APIKEY -X GET \
'https://api.softlayer.com/rest/v3.1/SoftLayer_Virtual_Guest/{VSI_ID}?objectMask=mask[hostname,id,operatingSystem.softwareLicense.softwareDescription.longDescription,provisionDate,frontendNetworkComponents.speed,frontendNetworkComponents.primaryIpAddress,maxMemory,maxCpu,backendNetworkComponents.speed,backendNetworkComponents.primaryIpAddress,allowedNetworkStorage,blockDevices.diskImage,localDiskFlag,blockDevices.bootableFlag,blockDevices.device,blockDevices.diskImage.capacity,blockDevices.diskImage.diskImageStorageGroup,blockDevices.diskImage.importedDiskType,blockDevices.diskImage.localDiskFlag,blockDevices.diskImage.description]'
```
{: pre}

Example output:

You can find the portable storages under **blockDevices** > **bootableFlag:0**. Use the **Description** value to map the respective portable storage volume.

```sh
{"hostname":"virtualserver01","id":154195996,"maxCpu":2,"maxMemory":4096,"provisionDate":"2025-11-06T07:43:52-06:00","allowedNetworkStorage":[{"accountId":1041833,"capacityGb":20,"createDate":"2025-10-30T02:21:22-07:00","guestId":null,"hardwareId":null,"hostId":null,"id":723195182,"nasType":"ISCSI","serviceProviderId":1,"storageTypeId":"7","upgradableFlag":true,"username":"IBM02SEL1041833-909","serviceResourceBackendIpAddress":"192.0.2.10","serviceResourceName":"Storage Type 02 Block Aggregate stbf-dal1303g"},{"accountId":1041833,"capacityGb":20,"createDate":"2025-10-21T07:06:09-07:00","guestId":null,"hardwareId":null,"hostId":null,"id":721245700,"nasType":"NAS","notes":"  Automation Storage Test - SMT - MOUNT \"\"","serviceProviderId":1,"storageTypeId":"13","upgradableFlag":true,"username":"IBM02SEV1041833_935","serviceResourceBackendIpAddress":"fsf-dal1301j-fz.adn.networklayer.com","serviceResourceName":"Storage Type 02 File Aggregate stff-dal1301j"}],"backendNetworkComponents":[{"speed":100,"primaryIpAddress":"192.0.2.10"}],"blockDevices":[{"bootableFlag":1,"device":"0","diskImage":{"capacity":100,"description":"virtualserver01.ibmcloud.private","localDiskFlag":true}},{"bootableFlag":0,"device":"1","diskImage":{"capacity":2,"description":"154195996-SWAP","localDiskFlag":true}},{"bootableFlag":0,"device":"2","diskImage":{"capacity":100,"description":"virtualserver01 - Disk 2","localDiskFlag":true}},{"bootableFlag":0,"device":"4","diskImage":{"capacity":10,"description":"virtualserver01 - Disk 3","localDiskFlag":true}},{"bootableFlag":0,"device":"7","diskImage":{"capacity":64,"description":"virtualserver01 - Metadata","localDiskFlag":true}}],"frontendNetworkComponents":[{"speed":100,"primaryIpAddress":"192.0.2.11"}],"localDiskFlag":true,"operatingSystem":{"hardwareId":null,"id":91776354,"manufacturerLicenseInstance":"","softwareLicense":{"id":80628,"softwareDescriptionId":3228,"softwareDescription":{"longDescription":"Ubuntu 24.04-64 Minimal for VSI"}}}}%
```
{: pre}

### Showing local and portable volume details from Mpath CLI example
{: #show-local-and-portable-volume-details-from-mpath-cli-example}

1. Get Mpath details of the file system mounted on a virtual server instance

    For this example, assume that the file system mounted on `/mnt/portableStorage`.

    ```sh
    lsblk -f
    ```
    {: pre}

    Example output:

    ```sh
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
    {: screen}

    Refer to the column `NAME` to get the block device name on which the file system exists. In this case, xvde contains a file system that is mounted on `/mnt/portableStorage`.

2. Get the block device number.

     Block device numbers start at 0 and correspond to disks in alphabetical order. For example, xvda is 0, xvdb is 1, xvdc is 2, and so on.

     In this case, xvde is mapped to device number 4.

3. Map the device number to the description.

     Get the `VSI_ID` to which the portable storage volume is attached. Then, run the following API call to get the description of block devices attached to the virtual server instance.

     ```sh
     curl -g -u $SL_USER:$SL_APIKEY -X GET 'https://api.softlayer.com/rest/v3.1/SoftLayer_Virtual_Guest/{VSI_ID}.json?objectMask=mask[hostname,id,operatingSystem.softwareLicense.softwareDescription.longDescription,provisionDate,frontendNetworkComponents.speed,frontendNetworkComponents.primaryIpAddress,maxMemory,maxCpu,backendNetworkComponents.speed,backendNetworkComponents.primaryIpAddress,allowedNetworkStorage,blockDevices.diskImage,localDiskFlag,blockDevices.bootableFlag,blockDevices.device,blockDevices.diskImage.capacity,blockDevices.diskImage.diskImageStorageGroup,blockDevices.diskImage.importedDiskType,blockDevices.diskImage.localDiskFlag,blockDevices.diskImage.description]'
     ```
     {: pre}

     Under **blockDevices**, you can find a list of all attached block devices. Map the **device** field value to the number from previous step. Then, check the **diskImage** field to get information about the **description** and **capacity** of the portable storage volume.

    Example output:

    ```sh
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
      "frontendNetworkComponents":
       [
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
    {: screen}

In this case, the device number is 4. Therefore, the portable storage volume that is mapped to the `/mnt/portableStorage` mount point is `virtualserver01 - Disk 3` with a capacity of 10 GB.

### Find a mount point in the virtual server instance for a particular local/portable storage volume example
{: #find-mount-point-from-portable-volume-details-cli-example}

1. Find the device number of the local/portable storage volume that is attached to the virtual server instance. The **diskImage** field under **blockDevices** contains details of the portable storage volume including the device number. For more information, see [Showing local and portable volume details](#showing-local-and-portable-volume-details-from-mpath-cli-example).
2. Map the device number to the block device name. Block device numbers start at 0 and correspond to disks in alphabetical order. For example, xvda is device number 0, xvdb is 1, and xvdc is 2.
3. Map the block device names to the mount point. In the following example, a file system exists on xvde (see the `NAME` column) for which the mount point is `/mnt/portableStorage` (see the `MOUNTPOINTS` column).

  Example output:

  ```sh
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
  {: screen}
