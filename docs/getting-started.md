# Getting started

This chapter describes how to setup a base XenServer system, running a DDK Virtual Machine (VM), for examining the examples provided in this document, and for use in the development of supplemental packs.
Partners who wish to construct supplemental packs as part of their own build systems should consult the appropriate section, later in this document.

The high-level process of setting up a DDK VM to create a supplemental pack is:

1.  Obtain matching XenServer product and DDK build ISOs.

1.  Install XenServer onto a host server.

1.  Install the XenCenter administrator console onto a Windows-based machine.

1.  Use XenCenter to import the DDK onto the XenServer host as a new virtual machine.

## Installing XenServer

Installing XenServer only requires booting from the CD-ROM image, and answering a few basic questions.
After setup completes, take note of the host IP address shown, as it is required for connection from XenCenter.

> **Note**
>
> Intel VT or AMD-V CPU support is required to run Windows guests, but is not required in order to use the DDK, nor for testing drivers in the XenServer control domain (dom0).

## Installing XenCenter

XenCenter, the XenServer administration console, must be installed on a separate Windows-based machine.
Inserting the XenServer installation CD will run the XenCenter installer automatically.
Once installed, the XenCenter console will be displayed with no servers connected.

## Connect XenCenter to the XenServer host

Within XenCenter select the **Server &gt; Add** menu option and supply the appropriate host name/IP address and password for the XenServer host.

Select the newly connected host in the left-hand tree view.

## Importing the DDK VM through XenCenter

> **Note**
>
> You can also import the DDK directly on the host using the **xe** Command Line Interface (CLI).

-  Insert the DDK CD into the CD-ROM drive of the machine running XenCenter.

-  On the **VM** menu, select the **Import** option. The VM Import Wizard is displayed.

-  Click **Browse** and on the **Files of type** drop-down list, select **XenServer Virtual Appliance Version 1 (ova.xml)**.

-  Navigate to the DDK CD-ROM and select the ova.xml file within the DDK directory.

-  Click **Next** to use the defaults on the Home Server and Storage pages.

-  On the Network page, add a virtual network interface to the VM.

-  Finish the VM Import Wizard.

The DDK VM will be started automatically.

## Importing the DDK VM using the CLI

The DDK VM can also be imported directly on the XenServer host using the xe CLI and standard Linux commands to mount the DDK ISO.

-  Mount the DDK ISO and import the DDK VM:

    ```bash
    mkdir -p /mnt/tmp
    mount <path_to_DDK_ISO>/ddk.iso /mnt/tmp –o loop,ro
    xe vm-import filename=/mnt/tmp/ddk/ova.xml
    ```

    The universally unique identifier (UUID) of the DDK VM is returned when the import completes.

-  Add a virtual network interface to the DDK VM:

    ```bash
    xe network-list
    ```

    Note the UUID of the appropriate network to use with the DDK VM, typically this will be the network with a `name-label` of `Pool-wide network associated with eth0`.

    ```bash
    xe vif-create network-uuid=<network_uuid> vm-uuid=<ddk_vm_uuid> device=0
    ```

    > **Note**
    >
    > Use tab completion to avoid entering more than the first couple of characters of the UUIDs.

-  Start the DDK VM:

    ```bash
    xe vm-start uuid=<ddk_vm_uuid>
    ```

## Using the DDK VM

Select the DDK VM in the left pane and then select the Console tab in the right pane to display the console of the DDK VM to provide a terminal window in which you can work.

The DDK VM is Linux-based so you are free to use other methods such as ssh to access the DDK VM. You can also access the DDK VM console directly from the host console.

## Adding Extra Packages to the DDK VM

The DDK is built to be as close as possible to the XenServer control domain (Dom0).
This means that only a small number of extra packages are present in the DDK (to enable the compilation of kernel modules) as compared to Dom0.
In some cases, partners who wish to use the DDK as a build environment may wish to add extra packages (e.g. NIS authentication) to the DDK.

Because the DDK (and Dom0) are based on CentOS, any package that is available for that distribution can be installed into the DDK, using the Yum package manager.
However, it is necessary to explicitly enable the CentOS repositories to allow such installation.
Package installation must therefore be carried out using the command:

    yum install <packageName>

## Accessing the DDK VM console from the host console

The DDK VM text console can be accessed directly from the XenServer host console instead of using XenCenter.
Note that using this method disables access to the DDK console from XenCenter.

-  While the DDK VM is shut down, disable VNC access on the VM record:

        xe vm-param-set uuid=<ddk_vm_uuid> other-config:disable_pv_vnc=1

-  Start the VM

        xe vm-start uuid=<ddk_vm_uuid>

-  Retrieve the underlying domain ID of the VM:

        xe vm-list params=dom-id uuid=<ddk_vm_uuid> --minimal

-  Connect to the VM text console:

        /usr/lib/xen/bin/xenconsole <dom-id>

-  When complete, exit the xenconsole session using **CTRL-\]**

For more information on using with the xe CLI please see the *XenServer Administrator's Guide*, available online.