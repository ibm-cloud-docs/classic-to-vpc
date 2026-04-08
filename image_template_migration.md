---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-08"

keywords: migrate image template, image template, import image to vpc infrastructure, migrate virtual server, migrate instance

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# Migrating an image template to VPC
{: #migrate-image-template-to-vpc}

You can migrate a virtual server instance from the Classic infrastructure to {{site.data.keyword.vpc_full}} by capturing its image. You can also create an image template from the instance and import it in the VPC environment as a [custom image](/docs/vpc?topic=vpc-planning-custom-images).
{: shortdesc}

Capturing a virtual server instance image from the Classic infrastructure isn't supported for LinuxONE (s390x processor architecture).
{: note}

## Before you begin
{: #before-migrating-virtual-server-from-classic}

Before you begin, make sure that the following prerequisites are fulfilled.

- You must have an instance of {{site.data.keyword.cos_full}} available.
- You must create an authorization so that the Image Service for VPC can access images in {{site.data.keyword.cos_full_notm}}. For more information, see [Granting access to {{site.data.keyword.cos_full_notm}} to import images](/docs/vpc?topic=vpc-object-storage-prereq).

## Migrating an image template from the Classic infrastructure
{: #migrate-vsi-from-classic-to-vpc-on-classic-task}

Complete the following steps to migrate an image template that is associated with a virtual server in the Classic infrastructure to the {{site.data.keyword.vpc_short}} infrastructure. When the custom image is available in {{site.data.keyword.vpc_short}}, you can use it to create a virtual server.

The following list is an overview of the migration steps.

1. For the virtual server that you want to migrate to {{site.data.keyword.vpc_short}} infrastructure, create an image template. The image template must be created before you proceed with the following migration steps.
2. From the image template, provision a virtual server that you want to use as the backup instance.
3. Customize the backup virtual server to make sure that it meets the requirements to deploy in {{site.data.keyword.vpc_short}}.
4. Create an image template of your modified virtual server.
5. Export the image template to {{site.data.keyword.cos_full_notm}}.
6. Import the custom image to the {{site.data.keyword.vpc_short}} infrastructure.
7. Use the custom image to create a virtual server in {{site.data.keyword.vpc_short}}.

If you already have an image template that matches all the requirements to deploy in {{site.data.keyword.vpc_short}}, you can proceed to step 5.

### Identifying the virtual server to migrate and create an image template
{: #migrate-create-template}
{: #step-1}

You can create an image template from a virtual server in the Classic infrastructure that you want to migrate to the {{site.data.keyword.vpc_short}} infrastructure. The image template captures an image of the existing virtual server. Make sure that you understand the following information about image templates.

* Only image templates with a single primary boot volume (or disk) and associated files can be imported to {{site.data.keyword.vpc_short}} infrastructure.
* The image template includes the operating system on the primary boot disk and the items that you installed, such as PHP or Python. The image template can be between 10 GB and 250 GB of data. Images under 10 GB are rounded up to 10 GB.
* When you use the imported custom image in {{site.data.keyword.vpc_short}} to create a virtual server, you can select a new profile, assign an SSH key, specify user data, and configure network interfaces.

Secondary disks and their associated files for an image template aren't supported when an image template is imported as a custom image to {{site.data.keyword.vpc_short}}.
{: important}

Complete the following steps to create an image template for the virtual server that you want to migrate.

1. From the dashboard in [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/){: external}, create an image template by clicking **Navigation menu** ![Menu icon](../../icons/icon_hamburger.svg) > **Infrastructure** > **Classic Infrastructure** > **Devices** > **Device List**.
2. Click the virtual server that you want to use.
3. From the **Actions** menu, select **Create image template**. Make sure to name it something you can easily recognize. For more information, see [Creating an image template](/docs/image-templates?topic=image-templates-creating-an-image-template).

### Locating the image template and provisioning a virtual server
{: #migrate-locate-template}
{: #step-2}

From the image template that you created, provision a virtual server. This instance is a backup. You can customize this backup virtual server to meet the requirements of {{site.data.keyword.vpc_short}}.

Complete the following steps to create a new virtual server from the image template.

1. Click **Devices** > **Manage** > **Images** to locate the image template that you created.
2. Provision a virtual server from the image template by clicking the Actions icon ![Actions icon](../icons/action-menu-icon.svg) for the image template and selecting **Order Public virtual server instance**.

### Customizing the virtual server for VPC
{: #migrate-customize-image-vpc}
{: #step-3}

Complete the following customizations to prepare your image for the {{site.data.keyword.vpc_short}} infrastructure. Some of the customization requirements might already be met on your Classic infrastructure virtual server instance.

#### Customizing a Linux&reg; virtual server
{: #customize-linux-instance}

Follow the instructions in [Creating a Linux custom image](/docs/vpc?topic=vpc-create-linux-custom-image) to customize on your Linux instance. Your instance needs to meet the following requirements.
   * The following arguments are present on the kernel command line: `nomodeset`, `nofb`, `vga=normal`, `console=ttyS0`.
   * [Virtio drivers](/docs/vpc?topic=vpc-create-linux-custom-image#virtio-drivers) are installed, plus any code that is needed by Virtio.
   * Your image is [cloud-init enabled](/docs/vpc?topic=vpc-create-linux-custom-image#cloud-init). If your instance is already using cloud-init, then you need to reset its cloud-init state. Follow the [cloud-init clean instructions](https://cloudinit.readthedocs.io/en/latest/reference/cli.html#clean){: external} to remove all cloud-init artifacts and allow it to rerun all stages on bootup.
   * For any auxiliary storage volumes that are mounted, you must include the _fstab_ entry `nofail`.

#### Customizing a Windows&reg; instance
{: #customize-windows-instance}

Complete the following customizations on your Windows virtual server to prepare the image for {{site.data.keyword.vpc_short}}.

1. Use remote desktop to access your Classic Windows server.
2. Download and install the Windows VirtIO drivers in this server. The `virtio-win` driver files can be taken from an existing {{site.data.keyword.vpc_short}} virtual server by using the following steps.

   Obtain the drivers from a licensed RHEL version 8 or 9 instance because drivers obtained from {{site.data.keyword.redhat_full}} are certified by Microsoft. The minimum recommended {{site.data.keyword.redhat_notm}} virtio-win package version is `virtio-win-1.9.24`. However, using the most recent package is best.
   {: note}

   The {{site.data.keyword.redhat_notm}} `virtio-win-1.9.24` ISO contains the following specific driver versions:

   ```text
   100.84.104.19500 oem10.inf \vioprot.inf_amd64_af0659efdaba9e4b\vioprot.inf
   100.90.104.21400 oem11.inf \viofs.inf_amd64_c6f785e21f3f6f80\viofs.inf
   100.85.104.20200 oem12.inf \viogpudo.inf_amd64_b19dcf9947e73e5a\viogpudo.inf
   100.85.104.19900 oem13.inf \vioinput.inf_amd64_4505a789e17b5f89\vioinput.inf
   100.81.104.17500 oem14.inf \viorng.inf_amd64_ef304eab276a3e61\viorng.inf
   100.85.104.19900 oem15.inf \vioser.inf_amd64_cb4783c018c10eba\vioser.inf
   100.90.104.21500 oem2.inf  \vioscsi.inf_amd64_02a46a7a223648d1\vioscsi.inf
   100.90.104.21500 oem3.inf  \viostor.inf_amd64_520417bbc533faba\viostor.inf
   100.85.104.20700 oem4.inf  \balloon.inf_amd64_afa8c93081df5458\balloon.inf
   100.90.104.21400 oem5.inf  \netkvm.inf_amd64_0efff05c07fcee39\netkvm.inf
   100.85.104.19900 oem6.inf  \pvpanic.inf_amd64_b7028360ef636f8b\pvpanic.inf
   10.0.0.21000     oem9.inf  \qxldod.inf_amd64_6199f9ecf2339133\qxldod.inf
   ```
   {: screen}

   1. Obtain the required virtio-win drivers by provisioning or accessing an existing RHEL virtual server in {{site.data.keyword.vpc_short}}.
   2. On the RHEL virtual server that you provisioned in {{site.data.keyword.vpc_short}}, install the `virtio-win` package by running the following command. In this example, the virtio-win package is installed on RHEL version 8.

   ```sh
   yum install virtio-win
   ```
   {: pre}

   The output looks like the following example.

   ```text
   Installed: virtio-win-1.9.24-2.el8_5.noarch
   ```
   {: screen}

   3. Access the virtio-win ISO in the `/usr/share/virtio-win` directory.

   ```sh
   cd /usr/share/virtio-win/
   ```
   {: pre}

   4. Use the WinSCP utility to copy the `virtio-win` ISO file, such as `virtio-win-1.9.24.iso`, from the RHEL VPC server to the Classic Windows server.
   5. Locate the downloaded ISO and double-click it to mount it.
   6. From the mounted ISO, run `virtio-win-guest-tools.exe` and complete the installation.
   7. Install and configure Cloudbase-Init from [Cloudbase-Init installation package](https://cloudbase.it/cloudbase-init/#download){: external}. If the instance is already Cloudbase-Init enabled, then you need to review the following steps and finally reset the Cloudbase-Init state.
   8. Modify the `cloudbase-init.conf` file (`C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\cloudbase-init.conf`) to match the following values. Don't remove any other content from the file.

   ```text
   [DEFAULT]
   #  "cloudbase-init.conf" is used for every boot
   config_drive_types=vfat,iso
   config_drive_locations=hdd,partition
   activate_windows=true
   kms_host=kms.adn.networklayer.com:1688
   mtu_use_dhcp_config=false
   real_time_clock_utc=false
   bsdtar_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\bsdtar.exe
   mtools_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\
   debug=true
   log_dir=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log\
   log_file=cloudbase-init.log
   default_log_levels=comtypes=INFO,suds=INFO,iso8601=WARN,requests=WARN
   local_scripts_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\
   metadata_services=cloudbaseinit.metadata.services.configdrive.ConfigDriveService,
   # enabled plugins - executed in order
   plugins=cloudbaseinit.plugins.common.mtu.MTUPlugin,
             cloudbaseinit.plugins.windows.ntpclient.NTPClientPlugin,
             cloudbaseinit.plugins.windows.licensing.WindowsLicensingPlugin,
             cloudbaseinit.plugins.windows.extendvolumes.ExtendVolumesPlugin,
             cloudbaseinit.plugins.common.userdata.UserDataPlugin,
             cloudbaseinit.plugins.common.localscripts.LocalScriptsPlugin
   ```
   {: screen}

   If you plan to bring your own license for your custom image, remove the following lines from the `cloudbase-init.conf` file.

   ```text
   activate_windows=true
   kms_host=kms.adn.networklayer.com:1688
   ```
   {: screen}

3. Modify the `cloudbase-init-unattend.conf` file (`C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\cloudbase-init-unattend.conf`) to match the following values. Don't remove any other content from the file.

   ```text
   [DEFAULT]
   #  "cloudbase-init-unattend.conf" is used during the Sysprep phase
   username=Administrator
   inject_user_password=true
   first_logon_behaviour=no
   config_drive_types=vfat
   config_drive_locations=hdd
   allow_reboot=false
   stop_service_on_exit=false
   mtu_use_dhcp_config=false
   bsdtar_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\bsdtar.exe
   mtools_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\
   debug=true
   log_dir=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log\
   log_file=cloudbase-init-unattend.log
   default_log_levels=comtypes=INFO,suds=INFO,iso8601=WARN,requests=WARN
   local_scripts_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\
   metadata_services=cloudbaseinit.metadata.services.configdrive.ConfigDriveService,
   # enabled plugins - executed in order
    plugins=cloudbaseinit.plugins.common.mtu.MTUPlugin,
             cloudbaseinit.plugins.common.sethostname.SetHostNamePlugin,
             cloudbaseinit.plugins.windows.createuser.CreateUserPlugin,
             cloudbaseinit.plugins.windows.extendvolumes.ExtendVolumesPlugin,
             cloudbaseinit.plugins.common.setuserpassword.SetUserPasswordPlugin,
             cloudbaseinit.plugins.common.localscripts.LocalScriptsPlugin
   ```
   {: screen}

4. Modify the `Unattend.xml` file (`C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\Unattend.xml`) and set the `PersistAllDeviceInstalls` value to `false`.
5. Run `Sysprep` by using the following command from the command prompt.

   ```sh
   C:\Windows\System32\Sysprep\Sysprep.exe /oobe /generalize /shutdown "/unattend:C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\Unattend.xml"
   ```
   {: pre}

   This command shuts down the system.
   {: important}

6. If the instance is already Cloudbase-Init enabled, then you need to reset the Cloudbase-Init state and artifacts. The reset will run the first boot steps again.
   1. Stop the cloudbase-init service

   ```sh
   net stop cloudbase-init
   ```
   {: pre}

   2. Delete cached instance data:

   ```powershell
   Remove-Item "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\cache" -Recurse -Force
   Remove-Item "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log" -Recurse -Force
   Remove-Item "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\run" -Recurse -Force
   ```
   {: pre}


   3. (Optional) Delete persistent network configuration if present:

   ```powershell
   Remove-Item "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\networks.json" -Force
   ```
   {: pre}

   4. Restart the service:

   ```powershell
   net start cloudbase-init
   ```
   {: pre}

### Creating an image template of your customized virtual server
{: #migrate-new-image-template}
{: #step-4}

When customization is complete on your backup virtual server, create a new image template by using the following steps:

1. From the Dashboard in [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/){: external}, click **menu** ![Menu icon](../../icons/icon_hamburger.svg) > **Infrastructure** > **Classic Infrastructure** > **Devices** > **Device List** to create an image template.
2. Click the backup virtual server that you previously customized.
3. From the **Actions** menu, select **Create Image Template**. The image template must be created before you can proceed with the following steps.

### Exporting the image template to IBM Cloud Object Storage
{: #migrate-export-template}
{: #step-5}

To export the image template that you created from the modified virtual server to {{site.data.keyword.cos_full_notm}}, complete the following steps.

1. Locate the new image template that you created on the **Image templates** page by clicking **Devices** > **Manage** > **Images**.
2. From the **Image Templates** page, click the ellipsis **...** for the image template that you want to export and select **Export to IBM Cloud Object Storage**. For more information, see [Exporting an image to {{site.data.keyword.cos_full_notm}}](/docs/image-templates?topic=image-templates-exporting-an-image-to-ibm-cloud-object-storage).

### Importing the custom image to the VPC infrastructure
{: #migrate-import-image}
{: #step-6}

1. In the {{site.data.keyword.cloud_notm}} console, click **Navigation menu** icon ![Menu icon](../icons/icon_hamburger.svg) > **Infrastructure** ![VPC icon](../../icons/vpc.svg) > **Compute** > **Images**.
1. On the **Custom images** tab, click **Import Custom Image**. For more information, see [Importing a custom image](/docs/vpc?topic=vpc-importing-custom-images-vpc).

### Using a custom image to create a virtual server in VPC
{: #migrate-create-virtual-server}
{: #step-7}

After the imported image is available on the **Custom images** tab of the **Images for VPC** page, you can use it to create a virtual server instance in the {{site.data.keyword.vpc_short}} infrastructure.

1. On the **Custom images** tab, find the name of the custom image that you imported, click Actions ![More Actions icon](../icons/action-menu-icon.svg), and select **New virtual server**.
2. In [{{site.data.keyword.cloud_notm}} console](https://console.cloud.ibm.com){: external}, go to the **Navigation menu** icon ![menu icon](../icons/icon_hamburger.svg) > **Infrastructure** ![VPC icon](../../icons/vpc.svg) > **Compute** > **Virtual server instances**.
3. On the **Virtual server instances** page, click actions ![More Actions icon](../icons/action-menu-icon.svg). Stop and then start the virtual server before you access it.
4. Create inbound and outbound security groups to give access to the RDP traffic port 3389. For more information, see [Setting up a security group for your resource](/docs/vpc?topic=vpc-configuring-the-security-group&interface=ui).
5. To generate a password to access to Windows and RDP with a floating IP, see [Connecting to Windows instances](/docs/vpc?topic=vpc-vsi_is_connecting_windows).
