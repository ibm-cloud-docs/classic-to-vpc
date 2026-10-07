---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: migration, migrate, migrating, migrate data, data migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating data from IBM Cloud classic infrastructure to VPC
{: #data-migration-classic-to-vpc}

This guide shows you how to connect your {{site.data.keyword.cloud}} classic infrastructure to {{site.data.keyword.vpc_full}} and migrate your data, especially for block or file volumes, as part of your migration journey. While many different data migration tools are available, the guidance on this page uses `rsync`. `rsync` is an open source utility that provides fast, incremental file transfer between two devices. It is available for both Linux&reg; and Windows&reg; platforms.

For more information, see [rsync documentation](https://linux.die.net/man/1/rsync){: external}.

Copying 1,000 GB of data by using `rsync` can take anywhere from 2.5 hours to more than 12 hours. The total time depends on several factors, such as the size of the files, disk performance, type of network connection, and overall network usage. Large files usually copy faster (around 3 hours at 100 MB/s), while copying millions of small files can take 12 hours or even several days due to higher processing overhead.

Block Storage for Classic and File Storage for Classic volumes typically range from 20 GB to 12 TB. Some customers who are allowlisted can use larger volumes, up to 16 TB for Block Storage and 24 TB for File Storage.

You can also use `dcfldd` to migrate raw block devices from Linux platforms. `dcfldd` is an enhanced version of GNU `dd` with additional features that are useful for forensics and security.

You can use other data migration tools instead of `rsync` or `dcfldd` to move your data to {{site.data.keyword.vpc_short}}.
{: note}

## Before you begin
{: #before-you-begin-data-migration}

Before you begin migrating your data, review the following requirements and considerations:

- You need an existing VPC environment.
- You are familiar with the storage capabilities and differences between Classic and VPC.
- Back up your data before you begin the migration process.

## Create IBM Cloud Transit Gateway and establish a connection between Classic and VPC
{: #create-ibm-cloud-transit-gateway-data-migration}

Your Classic infrastructure and {{site.data.keyword.vpc_short}} infrastructure need to be able to reach each other. You can achieve this in many ways, such as public interfaces, VPN, and Transit Gateway. If you need to move data from a few servers, you might want to use a public interface. However, if you have a large amount of data from different sources or large datasets, use {{site.data.keyword.tg_full_notm}}, which uses {{site.data.keyword.cloud_notm}} to interconnect Classic and VPC.

Before you create your {{site.data.keyword.tg_full_notm}}, review the following requirements and considerations:

- Ensure that Virtual Router Forwarding (VRF) is enabled on the classic infrastructure.
- Ensure that the IP network spaces don't overlap. Your VPC IP address must not be present in the classic infrastructure IP range.
- Ensure that your classic infrastructure data centers can connect to VPC. See [Transit Gateway-compatible classic data centers](/docs/transit-gateway?topic=transit-gateway-tg-locations#szr-table).
- Ensure that Classic and VPC access control lists (ACLs) and security groups are configured to allow ICMP and SSH/TCP connections.

* [Planning for {{site.data.keyword.tg_full_notm}}](/docs/transit-gateway?topic=transit-gateway-helpful-tips)
* [Ordering {{site.data.keyword.tg_full_notm}}](/docs/transit-gateway?topic=transit-gateway-ordering-transit-gateway)

If your compute resource has both public and private IP addresses, a host-level route must be added for the private connection to work correctly. Run the following command on your classic compute resources for your relevant operating system:

### Linux systems
{: #linux-systems-add-route}

```sh
ip route add <destination_network> via <Gateway_address> dev <private_ethernet_interface>
```
{: pre}

### Windows systems
{: #windows-systems-add-route}

```sh
route ADD <destination_network> MASK <subnet_mask> <gateway_ip>
```
{: pre}

## Installing `rsync`
{: #install-rsync}

Install `rsync` to enable file synchronization between systems during data migration.

### Linux systems
{: #linux-systems-install-rsync}

On most Linux systems, `rsync` is already installed. To verify whether `rsync` is installed, run the `rsync` command. If it isn't installed, complete the following steps for your relevant operating system.

#### Ubuntu
{: #install-rsync-ubuntu}

1. Make sure that you are logged in as the root user.
2. Run `sudo apt-get install rsync`.

#### Debian
{: #install-rsync-debian}

1. Log in as root.
2. Run `apt-get update`.
3. Run `apt-get install rsync`.

#### RHEL
{: #install-rsync-rhel}

1. Log in as root.
2. Run `yum install rsync`.

#### CentOS
{: #install-rsync-centos}

1. Log in as root.
2. Run `yum -y install rsync`.

### Windows systems
{: #windows-systems-install-rsync}

Complete the following steps to install `rsync` on both source and destination Windows systems.

1. Go to the [Cygwin installation page](https://cygwin.com/install.html){: external}, download `setup-x86_64.exe`, and run the installer.
2. Follow through all of the steps until you see a list of all Linux packages.
3. Select `rsync` (in the net category).
4. Select OpenSSH (in the net category).
5. Click **Continue** and finish the installation.
6. Open the Cygwin terminal, type the `rsync` command, and press Enter.
7. If the output shows `rsync` command details with options, then it is installed.

## Installing OpenSSH
{: #install-openssh}

OpenSSH enables data transfer from source to destination over an SSH connection.

### Linux systems
{: #linux-systems-openssh}

OpenSSH is installed by default on Linux systems. No additional installation steps are required.

### Windows systems
{: #windows-systems-openssh}

#### Windows Server 2019, 2022, and 2025
{: #windows-2019-2022-2025-openssh}

For the official installation guide, see [Installation of OpenSSH for Windows](https://github.com/MicrosoftDocs/windowsserverdocs/blob/main/WindowsServerDocs/administration/OpenSSH/OpenSSH_Install_FirstUse.md){: external}.

1. Run PowerShell as an Administrator.
2. Run the following cmdlet to check whether OpenSSH is available:

    ```powershell
    Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
    ```
    {: pre}

    The command returns the following output if neither component is already installed:

    ```powershell
    Name  : OpenSSH.Client~~~~0.0.1.0
    State : NotPresent

    Name  : OpenSSH.Server~~~~0.0.1.0
    State : NotPresent
    ```
    {: screen}

3. On both the source and destination computers, run the following cmdlets to install the client and server:

    ```powershell
    # Install the OpenSSH client on the source computer
    Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

    # Install the OpenSSH server on the destination computer
    Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
    ```
    {: pre}

    Both commands return the following output:

    ```powershell
    Path          :
    Online        : True
    RestartNeeded : False
    ```
    {: screen}

4. On the destination system, start and configure the OpenSSH server for initial use:

    ```powershell
    # Start the sshd service
    Start-Service sshd

    # Confirm the firewall rule is configured. It should be created automatically by setup. Run the following to verify:
    if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue)) {
        Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
        New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
    } else {
        Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
    }
    ```
    {: pre}

#### Windows Server 2016
{: #windows-openssh-installation}

Windows Server 2016 is in extended support until January 12, 2027. Upgrade to Windows Server 2019 or 2022 for better security and support.
{: important}

For more information, see the following resources:

* [End of Support announcements for Windows Server 2016](/docs/vpc?topic=vpc-eos-os-considerations-intro#windows-server-2016-eom-eos-vpc)
* [Get started with OpenSSH Server for Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui&pivots=windows-server-2025){: external}
* [OpenSSH Server Configuration for Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration?source=recommendations){: external}

For the installation procedure, see [Installation of OpenSSH for Windows 2016](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH){: external}.

1. Run PowerShell as Administrator.
2. Run the following commands to install OpenSSH:

    ```powershell
    mkdir c:\openssh-install
    cd c:\openssh-install

    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

    Invoke-WebRequest -Uri "https://github.com/PowerShell/Win32-OpenSSH/releases/download/V8.6.0.0p1-Beta/OpenSSH-Win64.zip" -OutFile .\openssh.zip

    Expand-Archive .\openssh.zip -DestinationPath .\openssh\

    cd .\openssh\OpenSSH-Win64\

    powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1
    ```
    {: pre}

3. On the destination computer, run the following commands to open the firewall port and start the OpenSSH server:

    ```powershell
    New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

    Start-Service sshd
    ```
    {: pre}

## Generating SSH keys
{: #generate-ssh-keys}

This section describes how to generate SSH keys on various operating systems.

### Linux systems
{: #linux-systems-generate-ssh-keys}

1. Generate the private-public key pair by running `ssh-keygen -t rsa`.
2. Save the generated keys to `{user's home directory}/.ssh`.
3. Copy the public key to the destination server at `{user's home directory}/.ssh/authorized_keys`.

### Windows systems
{: #windows-systems-generate-ssh-keys}

1. Generate the private-public key pair by running `ssh-keygen -t rsa`.
2. Save the generated keys to `C:\Users\{username}\.ssh`.
3. Copy the public key to the destination server at `C:\Users\{username}\.ssh\authorized_keys`.

## Running the auto `rsync` script to migrate file system-based volumes
{: #run-rsync-script}

Download the auto `rsync` scripts from the [vpc-migration-tools repository](https://github.com/IBM-Cloud/vpc-migration-tools/tree/main/data-migration){: external}.

Run the script in a `screen` or `tmux` session.
{: note}

### Linux systems
{: #linux-systems-rsync-script}

1. Run `bash <auto_rsync_file_name>` and press Enter.
2. Enter the path of the source, for example: `/home/Demo-12Jan/12Jan_test1.txt`
3. Enter the path of the destination, for example: `/home/19Jan`
4. Enter the address of the target server, for example: `192.168.0.5`
5. Enter the target username, for example: `root`
6. (Optional) Enter custom options. For a list of custom options, see [rsync(1) - Linux man page](https://linux.die.net/man/1/rsync){: external}.
7. After you enter all the attributes, the `rsync` process starts.

### Windows systems
{: #windows-systems-rsync-script}

1. Download or copy the script file to the `/cygdrive/c/cygwin64/home/Administrator` directory on the source server.
2. Open the Cygwin terminal.
3. Run `cd /cygdrive/c/cygwin64/home/Administrator`.
4. Run `bash ./<auto_rsync_file_name>` and press Enter.
5. Enter the path of the source, for example: `/cygdrive/c/home/Demo-12Jan/12Jan-test1.txt`
6. Enter the path of the destination, for example: `/cygdrive/c/home/19Jan/`
7. Enter the address of the target server, for example: `192.168.0.5`
8. Enter the target username, for example: `Administrator`
9. (Optional) Enter custom options.
10. After you enter all the attributes, the `rsync` process starts.

## Migrating raw block devices by using `dcfldd`
{: #run-dcfldd}

To migrate raw block devices, use `dcfldd` to generate a local backup of the disk, transfer the backup files to the destination by using `rsync` and SSH, and then write back to a block device.

For a successful transfer, you need temporary storage that is equal to or larger than the block device to store the disk backup file locally.

Run the script in a `screen` or `tmux` session.
{: note}

### Linux systems
{: #linux-systems-dcfldd}

1. Run the following command to back up the disk:

    ```sh
    dcfldd if=<block-device-path> of=<output-file-path> hash=sha256 hashlog=<path-to-hashlog-file>
    ```
    {: pre}

    For example:

    ```sh
    dcfldd if=/dev/sda of=/mnt/temp-storage/sda-backup.img hash=sha256 hashlog=/mnt/temp-storage/sda-backup-hashlog
    ```
    {: pre}

    In this example, `/dev/sda` is the disk to be migrated.

2. Transfer the disk backup file and hash file to the destination computer by using the auto `rsync` script from the previous section, or run the following command directly:

    ```sh
    rsync -avP <output-file-path> <path-to-hashlog-file> root@<remote-host-ip>:<path-to-destination>
    ```
    {: pre}

3. On the destination computer, run the following command to restore the disk from the transferred backup file:

    ```sh
    dcfldd if=<output-file-path> of=<block-device-path> hash=sha256 hashlog=<path-to-hashlog-file>
    ```
    {: pre}

    For example:

    ```sh
    dcfldd if=/mnt/vpc-temp-storage/sda-backup.img of=/dev/sdb hash=sha256 hashlog=/mnt/vpc-temp-storage/sda-restore-hashlog
    ```
    {: pre}

    In this example, `/dev/sdb` is the disk to restore to.

4. Compare the generated hashlog files to verify data integrity:

    ```sh
    diff -q <backup-hashlog-file> <restore-hashlog-file>
    ```
    {: pre}
