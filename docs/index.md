# Citrix XenServer Supplemental Packs and the DDK Guide

Supplemental packs are used to modify and extend the functionality of a XenServer host, by installing software into the control domain, dom0.
For example, an OEM partner might wish to ship XenServer with a suite of management tools that require SNMP agents to be installed, or provide a driver that supports the latest hardware.
Users can add supplemental packs either during initial XenServer installation, or at any time afterwards.
Facilities also exist for OEM partners to add their supplemental packs to the XenServer installation repositories, in order to allow automated factory installations.

## Purpose of supplemental packs

Supplemental packs consist of a number of packages along with information describing their relationship to other packs.
Individual packages are in the Red Hat RPM file format, and must be able to install and uninstall cleanly on a fresh installation of XenServer.

Packs are created using the XenServer Driver Development Kit (DDK).
This has been extended to not only allow the creation of supplemental packs containing only drivers (also known as driver disks), but also packs containing userspace software to be installed into dom0.

Examples and tools are included in the XenServer DDK to help developers create their own supplemental packs.
However, for partners wishing to integrate pack creation into their existing build environments, only a few scripts taken from the DDK are necessary.

## Why a separate DDK?

XenServer is based on a standard Linux distribution, but for performance, maintainability, and compatibility reasons ad-hoc modifications to the core Linux components are not supported.
As a result, operations that require recompiling drivers for the Linux kernel require formal guidance from Citrix, which the DDK provides.
In addition, the DDK provides the necessary compile infrastructure to achieve this, whereas a XenServer installation does not.

XenServer integrates the latest device support from kernel.org on a regular basis to provide a current set of device drivers.
However, assuming appropriate redistribution agreements can be reached, there are situations where including additional device drivers in the shipping XenServer product, such as drivers not available through kernel.org, or drivers that have functionality not available through kernel.org, is greatly beneficial to joint customers of Citrix and the device manufacturer.
The same benefits can apply by supplying device drivers independent of the XenServer product.

In addition, components such as command line interfaces (CLIs) for configuring and managing devices are also very valuable to include in the shipped XenServer product.
Some of these components are simple binary RPM installs, but in many cases they are combined with the full driver installation making them difficult or impossible for
administrators to install into XenServer.
In either case including current versions of everything the administrator requires to use the device on XenServer in a supplemental pack provides significant value.

The DDK allows driver vendors to perform the necessary packaging and compilation steps with the XenServer kernel, which is not possible with the XenServer product alone.
Supplemental packs can be used to package up both drivers and userspace tools into one convenient ISO that can be easily installed by XenServer users.

## Benefits

Supplemental packs have a variety of benefits over and above partners producing their own methods for installing add-on software into XenServer:

-  Integration with the XenServer installer: users are prompted to provide any extra drivers or supplemental packs at installation time.
    In addition, on upgrade, users are provided with a list of currently installed packs, and warned that they may require a new version of them that is compatible with the new version of XenServer.

-  Flexibility in release cycles: partners are no longer tied to only releasing updates to their add-on software whenever new versions of XenServer are released.
    Instead, partners are free to release as often as they choose.
    The only constraint is the need to test packs on the newest version of XenServer when it is released.

-  Integration with Server Status Reports: supplemental pack metadata can include lists of files (or commands to be run) that should be collected when a Server Status Report is collected using XenCenter.
    Pack authors can choose to create new categories, or add to existing ones, to provide more user-friendly bug reporting.

-  Guarantee of integrity: supplemental packs are signed by the creator, allowing users to be certain of their origin.

-  Include formal dependency information: pack metadata can detail installation requirements such as which versions of XenServer the pack can be installed upon.

-  Inclusion in the Citrix Ready catalogue: partners whose supplemental packs meet certain certification criteria will be allowed to list their packs in the Citrix Ready online catalogue, thus increasing their visibility in the marketplace.
    Note that partners must become members of the program before their packs can be listed: the entry level category of membership is fee-free.

## What should (not) be in a supplemental pack?

Citrix recognises that partner organizations can contribute significant value to the XenServer product by building solutions upon it.
Examples include host management and monitoring tools, backup utilities, and device-specific firmware.
In many cases, *some* of these solutions will need to be hosted in the XenServer control domain, dom0, generally because they need privileged access to the hardware.

Whilst supplemental packs provide the mechanism for installing components into dom0, pack authors should try to install *as little as possible* using packs.
Instead, the majority of partner software should be placed into appliance virtual machines, which have the advantage that the operating system environment can be configured exactly as required by the software to be run in them.

The reasons for this stipulation are:

-  XenServer stability and QA: Citrix invests considerable resources in testing the stability of XenServer.
    Significant modifications to dom0 are likely to have unpredictable effects on the performance of the product, particularly if they are resource-hungry.

-  Supportability: the XenServer control domain is well-known to Citrix support teams.
    If it is heavily modified, dom0 becomes very difficult to identify whether the cause of the problem is a component of XenServer, or due to a supplemental pack.
    In many cases, customers may be asked to reproduce the problem on an unmodified version of XenServer, which can cause customer dissatisfaction with the organization whose pack has been installed.
    Similarly, when a pack author is asked to debug a problem perceived to be with their pack, having the majority of the components of the pack in an appliance VM of known/static configuration can significantly ease diagnosis.

-  Resource starvation: dom0 is limited in memory and processing power.
    If resource-hungry processes are installed by a supplemental pack, resource starvation can occur.
    This can impact both XenServer stability and the correct functioning of the supplemental pack.
    Note that Citrix does not advise increasing the number of dom0 vCPUs.

-  Security: XenServer dom0 is designed to ensure the security of the hosts that it is installed on to.
    Any security issues found in software that is installed into dom0 can mean that the host is open to compromise.
    Hence, the smaller the quantity of software installed into dom0 by a pack, the lower the likelihood that XenServer hosts will be compromised due to a flaw in the software of the pack.

Partners often ask whether supplemental packs can include heavy-weight software, such as the Java runtime environment, or a web server.
This type of component is not suitable for inclusion in dom0, and should instead be placed in an appliance VM.
In many cases, the functionality that is desired can be achieved using such an appliance VM, in conjunction with the Xen API.
Citrix can provide advice to partners in such cases.