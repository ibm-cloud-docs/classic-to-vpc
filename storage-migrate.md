---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-09"

keywords: migration, migrate, migrating, migrate data, data migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating data from IBM Cloud classic infrastructure to VPC
{: #data-migration-classic-to-vpc}

This guide shows you how to connect your {{site.data.keyword.cloud}} classic infrastructure to {{site.data.keyword.vpc_full}} and migrate your data, especially for block or file volumes, as part of your migration journey. While many different data migration tools are available, the guidance on this page uses `rsync`. `Rsync` is an open source utility that provides file transfer between two devices. It is available for both Linux&reg; and Windows&reg; platforms.

For more information, see [Rsync documentation](https://linux.die.net/man/1/rsync){: external}

Copying 1000 GB of data using `rsync` can take anywhere from 2.5 hours to more than 12 hours. The total time depends on several factors, such as the size of the files, disk performance, type of network connection, and overall network usage. Large files usually copy faster (around 3 hours at 100 MB/s), while copying millions of small files can take 12 hours or even several days due to higher processing overhead.
Block Storage for Classic and File Storage for Classic volumes typically range from 20 GB to 12 TB. Some customers who are allow‑listed can use larger volumes, up to 16 TB for Block Storage and 24 TB for File Storage.

You can also use `dcfldd` to migrate raw or block device from Linux platforms. `dcfldd` is an enhanced version of GNU dd with features useful for forensics and security.

You can use other data migration tools instead of `rsync` or `dcfldd` to move your data to {{site.data.keyword.vpc_short}}.
{: note}

## Before you begin
{: #before-you-begin-data-migration}

Before you begin migrating your data, review the following requirements and considerations:

- You need an existing VPC environment.
- You are aware of the storage capabilities and differences between Classic and VPC.
- Back up your data before you begin the migration process.

## Create IBM Cloud Transit Gateway and establish a connection between Classic and VPC
{: #create-ibm-cloud-transit-gateway-data-migration}
{: #step-1}

Your Classic infrastructure and {{site.data.keyword.vpc_short}} infrastructure need to be able to reach each other. You can achieve this in many ways, such as public interfaces, VPN, and transit gateway. If you need to move data from a few servers, you might want to use a public interface. However, if you have a large amount of data from different sources or large datasets, then you need to use {{site.data.keyword.tg_full_notm}}, which uses {{site.data.keyword.cloud_notm}} to interconnect between Classic and VPC.

Before you create your {{site.data.keyword.tg_full_notm}}, review the following requirements and considerations:

- Help ensure that the Virtual Router Forwarding (VRF) is enabled on the classic infrastructure.
- Help ensure that the IP network spaces don't overlap. Your VPC IP address must not be present in the classic infrastructure IP range.
- Help ensure that your classic infrastructure data centers are able to connect to VPC. [Transit-Gateway-compatible classic data centers](/docs/transit-gateway?topic=transit-gateway-tg-locations#szr-table)
- Help ensure the Classic and VPC access control list (ACL) and security group is configured to allow ICMP and SSH/TCP connection.

To create {{site.data.keyword.tg_full_notm}} and establish the connection between classic and VPC, review the following information:
* [Planning for {{site.data.keyword.tg_full_notm}}](/docs/transit-gateway?topic=transit-gateway-helpful-tips)
* [Ordering {{site.data.keyword.tg_full_notm}}](/docs/transit-gateway?topic=transit-gateway-ordering-transit-gateway)

If your compute resource has both public and private IP addresses, the host level route needs to be added for it to work properly. Run the following command on your classic compute resources for your relevant operating system:

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
{: #Step-2}

Install `rsync` to enable file synchronization between systems during data migration.

### Linux systems
{: #linux-systems-install-rsync}

On most Linux systems, `rsync` is already installed. To verify whether `rsync` in installed, run the `rsync` command. If it isn't installed, complete the following steps for your relevant operating system.

#### Ubuntu
{: #install-rsync-ubuntu}

1. You need to be the "root" user to install the package.
2. Run `sudo apt-get install rsync`.

#### Debian
{: #install-rsync-debian}

1. Log in with root.
2. Run `apt-get update`.

#### RHEL
{: #install-rsync-rhel}

1. Log in with root.
2. Run `yum install rsync`.

#### CentOS
{: #install-rsync-centos}

1. Log in with root.
2. Run `yum -y install rsync`.

### Windows systems
{: #windows-systems-install-rsync}

Complete the following steps to install `rsync` on both source and destination Windows system.

1. On the Cygwin (https://cygwin.com/install.html){: external} page, download `setup-x86_64.exe` and run the installer.
2. Follow through all of the steps until you see a list of all Linux packages.
3. Select `rsync` (in net category section).
4. Select OpenSSH (in net category section).
5. Click Continue and finish the installation.
6. Open the Cygwin terminal, type `rsync` command, and press enter.
7. If the output shows `rsync` command details with options, then it is installed.

## Installing OpenSSH
{: #install-openssh}
{: #Step-3}

OpenSSH can be used to enable transfer of data from source to destination over an SSH connection.

### Linux systems
{: #linux-systems-openssh}

If you have a Linux system, you don't need to install OpenSSH because it is installed by default.

### Windows systems
{: #windows-systems-openssh}

#### Windows 2019, 2022 and 2025
{: #windows-2019-2022-2025-openssh}

GitHub instructions on [Installation of OpenSSH for Windows](https://github.com/MicrosoftDocs/windowsserverdocs/blob/main/WindowsServerDocs/administration/OpenSSH/OpenSSH_Install_FirstUse.md){: external}

1. Run PowerShell as an Administrator.
2. Run the following cmdlet to make sure that OpenSSH is available:
    ```powershell
    Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
    ```

    The command returns the following output if neither is already installed:
    ```powershell
    Name  : OpenSSH.Client~~~~0.0.1.0
    State : NotPresent

    Name  : OpenSSH.Server~~~~0.0.1.0
    State : NotPresent
    ```

3. On both the source and destination computers, run the following cmdlets to install the client and server.

    ```powershell
    # Install the OpenSSH client on source computer
    Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

    # Install the OpenSSH server on destination computer
    Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
    ```

    Both commands return the following output:
    ```powershell
    Path          :
    Online        : True
    RestartNeeded : False
    ```

4. On the destination system, start and configure OpenSSH server for initial use

    ```powershell
    # Start the sshd service
    Start-Service sshd

    # Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify:
    if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue)) {
        Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
        New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
    } else {
        Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
    }
    ```

#### Windows 2016, 2019, 2022
{: #windows-openssh-installation}

The follow instructions are for Windows Server 2016, which is in extended support until January 12, 2027. It is recommended to upgrade to Windows Server 2019 or 2022 for better security and support.
For more information links for OpenSSH for windows: [End of Support (EOS) announcements](/docs/vpc?topic=vpc-eos-os-considerations-intro#windows-server-2016-eom-eos-vpc)
[Get started with OpenSSH Server for Windows] (https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui&pivots=windows-server-2025){: external}
[OpenSSH Server Configuration for Windows] (https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration?source=recommendations){: external}
{: important}

To install OpenSSH on your Windows 2016 system, review the following information:

GitHub instructions on [Installation of OpenSSH for Windows 2016](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH){: external}

1. Run PowerShell as administrator
2. Run the set of commands to install OpenSSH
    ```powerhshell
    mkdir c:\openssh-install
    cd c:\openssh-install

    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

    Invoke-WebRequest -Uri "https://github.com/PowerShell/Win32-OpenSSH/releases/download/V8.6.0.0p1-Beta/OpenSSH-Win64.zip" -OutFile .\openssh.zip

    Expand-Archive .\openssh.zip -DestinationPath .\openssh\

    cd .\openssh\OpenSSH-Win64\

    powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1
    ```

3. On the destination computer, run the following command to start OpenSSH server
    ```powershell
    New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

    Start-Service sshd
    ```

## Generating SSH keys
{: #generate-ssh-keys}
{: #Step-4}

This section describes how to generate SSH keys on various operating systems.

### Linux systems
{: #linux-systems-generate-ssh-keys}

1. Generate the private-public key by using the command `ssh-keygen -t rsa`
2. Save the generated keys to `{User’s home directory}/.ssh`
3. Copy the public key to the destination server by using the following path: `{user’s home directory}/.ssh/authorized_keys`

### Windows systems
{: #windows-systems-generate-ssh-keys}

1. Generate the private-public key by using the command `ssh-keygen -t rsa`
2. Save the generated keys to `C:\Users\{username}\.ssh`
3. Copy the public key to the destination server by using the following path: `C:\Users\{username}\.ssh\authorized_keys`

## Running auto `rsync` script to migrate file system-based volumes
{: #run-rsync-script}
{: #Step-5}

Download the auto `rsync` scripts from [tools](https://github.com/IBM-Cloud/vpc-migration-tools/tree/main/data-migration){: external}.

Running the script in a screen or tmux session is recommended.
{: note}

### Linux systems
{: #linux-systems-rsync-script}

1. Type `Bash <auto_rsync_file_name>` and press enter.
2. Enter the path of the source: `/home/Demo-12Jan/12Jan_test1.txt`
3. Enter the path of the destination: `/home/19Jan`
4. Enter the address of the target server: `192.168.0.5`
5. Enter the target username: `root`
6. (Optional) Enter custom options. For a list of custom options, see [rsync(1) - Linux man page](https://linux.die.net/man/1/rsync){: external}
7. Enter all the attributes and the `rsync` process starts.

### Windows systems
{: #windows-systems-rsync-script}

1. Download or copy and paste file to directory `/cygdrive/c/cygwin64/home/Administrator` of source server.
2. Open the Cygwin terminal.
3. Type `cd /cygdrive/c/cygwin64/home/Administrator`
4. Type `Bash ./<auto_rsync_file_name>` and press enter.
5. Enter the path of the source: `/cygdrive/c/home/Demo-12Jan/12Jan-test1.txt`
6. Enter the path of the destination: `/cygdrive/c/home/19Jan/`
7. Enter the address of the target server: `192.168.0.5`
8. Enter the target username: `Administrator`
9. (Optional) Enter custom options.
10. Enter all the attributes and the `rsync` process starts.

## To migrate raw or block devices by using `dcfldd`
{: #run-dcfldd}
{: #Step-6}

To migrate raw or block devices, you can use the `dcfldd` command to generate a local backup of the disk. Then, transfer the backup files over to a destination computer by using `rsync` and `SSH` and write back to a block device.

For a successful transfer, you need temporary storage equal to or larger than the block device to store the disk backup file locally.

Running the script in a screen or tmux session is recommended.
{: note}

### Linux systems
{: #linux-systems-dcfldd}

1. Run the following command:
    ```sh
    dcfldd if=<block-device-path> of=<output-file-path> hash=sha256 hashlog=<path-to-hashlog-file>

    example: dcfldd if=/dev/sda of=/mnt/temp-storage/sda-backup.img hash=sha256 hashlog=/mnt/temp-storage/sda-backup-hashlog
    ```
    `/dev/sda` in the preceding example is the disk to be migrated

2. Transfer the disk backup file and hash file over to the destination computer by using auto rsync script from the preceding step 5 OR run the following command:
    ```sh
    rsync -avP <output-file-path> <path-to-hashlog-file> root@RemoteHostIP:<path-to-destination>
    ```
3. On the destination computer, run the following command to recover the disk by using the transferred backup file:
    ```sh
    dcfldd if=<output-file-path> of=<block-device-path> hash=sha256 hashlog=<path-to-hashlog-file>

    example: dcfldd if=/mnt/vpc-temp-storage/sda-backup.img of=/dev/sdb hash=sha256 hashlog=/mnt/vpc-temp-storage/sda-restore-hashlog
    ```
    `/dev/sdb` is the disk to restore on.

4. Compare the generated hashlog files to verify integrity `diff -q <backup-hashlog-file> <restore-hashlog-file>`
