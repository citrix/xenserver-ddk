# Installing supplemental packs

You can install a supplemental pack at the same time as installing XenServer or you can install a supplemental pack on a running XenServer instance.

## At installation time

You can install supplemental packs while installing XenServer on a host in one of two ways:

1.  From a CD – during an interactive installation from local media you will be asked if you want to install any supplemental packs.
    Any number of packs can be installed in succession.

1.  From a Network repository – a HTTP, FTP or NFS repository can be expanded to include one or more supplemental packs.
    See the XenServer Installation Guide for instructions on how to extract a supplemental pack to a directory.

If a driver needs to be loaded from a supplemental pack before the XenServer installation can proceed (for example, to support a new RAID
controller), the user should press the F9 key when prompted by the installer.
The media containing the supplemental pack/driver disk can then be provided, and the installer will attempt to load the new driver.

The XenServer installer will allow the user to choose to verify a pack before it is installed.
The installer will check the contents of the pack against internal checksums produced when the pack was created.

## On a running host

Install a supplemental pack by using XenCenter or the xe CLI.

### Using XenCenter

To install a supplemental pack using XenCenter:

1.  Download the supplemental pack (`filename.iso`) to a known location on your computer.
    Supplemental packs are available to download from the XenServer downloads page.

1.  From the XenCenter menu, select **Tools** and then **Install Update**.

1.  Read the information on the **Before You Start** page and then select **Next** to continue

1.  On the **Select Update** page, click **Add** to add the supplemental pack.

1.  On the **Select Servers** page, select the XenServer host or the pool to which you would like to apply the supplemental pack and then click **Next**.
    This uploads the supplemental pack to the host or the pool’s default SR.

    > **Note**
    >
    > If the default SR in a pool is not shared, or does not have enough space, XenCenter tries to upload the supplemental pack to another shared SR with sufficient space.
    > If none of the shared SRs have sufficient space, the supplemental pack will be uploaded to each host's local storage.

1.  The **Upload** page displays the status of the upload.
    If there is not enough space on the SR, an error will be displayed.
    Click **More Info** for details and take necessary action to free up the space required for the upload.

1.  After the file is successfully uploaded, XenCenter performs a series of prechecks to determine whether the supplemental pack can be applied onto the selected servers and displays the result.
    Follow the on-screen recommendations to resolve any update prechecks that have failed.
    If you would like XenCenter to automatically resolve all failed prechecks, click **Resolve All**.

1.  Choose the **Update Mode**. Review the information displayed on the screen and select an appropriate mode.
    If you click **Cancel** at this stage, the **Install Update** wizard will revert the changes and removes the supplemental pack from the SR.

1.  Click **Install update** to proceed with the installation.
    The **Install Update** wizard shows the progress of the update, displaying the major operations that XenCenter performs while updating each host in the pool.

1.  When the supplemental pack has been installed, click **Finish** to close the wizard.

### Using CLI

A Supplemental Pack can be installed remotely using the xe CLI.

1.  Upload the update:

         xe update-upload file-name=<pack.iso>

    > **Note**
    >
    > The UUID of the pack is returned when the upload completes.

1.  Apply the pack:

         xe update-apply uuid=<pack_uuid>

    A pack can also be installed on the XenServer host by running the following command:

         xe-install-supplemental-pack <pack.iso>

    Example syntax for applying an Update package via the CLI:

        # xe update-upload file-name=test-hotfix-basic-1.iso
        320232df-7adb-4cbe-a7a3-8515240879e1
        # xe update-apply uuid=320232df-7adb-4cbe-a7a3-8515240879e1
        # xe update-list

        uuid ( RO): 320232df-7adb-4cbe-a7a3-8515240879e1
        name-label ( RO): test-hotfix-basic-1
        name-description ( RO): Simple basic hotfix with a single package, no guidance
        installation-size ( RO): 32
        hosts (SRO): 4fab6121-ea56-4c27-b4e3-b02aac6c2e28
        after-apply-guidance (SRO):

## Driver-specific considerations

The XenServer installer, when booted, loads kernel modules that are appropriate to the hardware it has detected.
If the newer version of a driver is needed in order for the installation to proceed, then the installer version of the module must be unloaded, and the new driver
loaded from a supplemental pack/driver disk.
The procedure for this is as follows:

1.  Reboot the host, leaving the XenServer installation CD-ROM in the drive.

1.  At the `boot:` prompt, type:

        shell

1.  You will now be presented with a command prompt. Type the following:

        rmmod <driver-name>

    Where &lt;*driver-name*&gt; is the name of the driver to be replaced.
    If this command succeeds (that is, if there are no error messages printed), the installer version of the driver has been unloaded.
    If error messages are presented, it is likely that other drivers depend on the driver you are attempting to unload.
    If this is the case, please contact Citrix support.

1.  Type

        exit

    or press Control+D on your keyboard, to return to the installer.

1.  In the installer, press the F9 key when prompted to provide the driver disk, which should now load correctly.