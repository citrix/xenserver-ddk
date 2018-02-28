# Testing and certification

XenServer uses a modified Linux kernel that is similar but not identical
to the kernel distributed by a popular Linux distribution. In contrast,
the XenServer control domain is currently based on a different
distribution. In addition, the 32 bit XenServer control domain kernel is
running above the 80K lines of code that are the 64 bit Xen hypervisor
itself. While Citrix is very confident in the stability of the
hypervisor, its presence represents a different software installation
than exists with the stock vendor kernel installed on bare hardware.

In particular, there are issues that may be taken for granted on an x86
processor, such as the difference between physical and device bus memory
addresses (for example **virt\_to\_phys()** as opposed to
**virt\_to\_bus()**), timing, and interrupt delivery which may have
subtle differences in a hypervisor environment.

For these reasons hardware drivers should be exposed to a set of testing
on the XenServer control domain kernel to ensure the same level of
confidence that exists for drivers in enterprise distribution releases.
Similarly, userspace software that is included in supplemental packs
must be tested comprehensively, to ensure that the assumptions it makes
about the environment in which it runs (for example, concerning the
presence of certain executables) are not invalidated.

The remainder of this section considers driver testing. Partners who
wish to release supplemental packs do not only contain drivers should
contact Citrix for advice. As a minimum, such pack authors should expect
to comprehensively test the functionality of the software being included
in the pack, as well as perform stress testing of the XenServer major
features, to ensure that none are impacted by the software in the pack.

## Testing scope

Assuming the driver in question has already undergone verification
testing on a Linux distribution very similar to the one used in the
XenServer control domain, a subset of the verification test suite with a
focus on representative tests is typically sufficient.

Some common areas of focus are:

-  *Installation verification.* Installation is now performed using an
    RPM, which may be different from how the driver is typically
    installed.

-  *CLI operation.* If a CLI is included, its operation is a key
    scenario and typically provides a good end-to-end exercising of all
    related code.

-  *Adapter configuration, non-volatile/flash RAM, or BIOS upgrades.*
    Any functions that access the physical hardware should be verified
    due to the presence of the Xen hypervisor and its role in
    coordination of hardware resources amongst virtual machines.

-  *Data integrity and fault injection.* Long-haul and/or stress-based
    data integrity verification tests to verify no data corruption
    issues exist. Basic fault injection tests such as cable
    un-plug/re-plug and timeout handling for verification of common
    error conditions.

-  *Coverage of a representative set of device models.* For example, if
    a Fibre Channel HBA driver supports a set of models that operate at
    either 2 Gb/s or 4 Gb/s, include one model each from the 2 Gb/s and
    4 Gb/s families for testing.

-  *Key hardware configuration variations.* Any hardware configurations
    that exercise the driver or related code in significantly different
    ways should be run, such as locally versus remotely attached
    storage.

## Running tests

Since the physical device drivers run in the XenServer control domain,
the majority of tests will also be run in the control domain. This
allows simple re-use of existing Linux-based tests.

To provide high-performance device I/O to guest domains, XenServer
includes synthetic device drivers for storage and networking that run in
a guest and communicate their I/O requests with corresponding back-end
drivers running in the control domain. The back-end drivers then issue
the I/O requests to the physical drivers and devices and manage
transmitting results and completion notifications to the synthetic
drivers. This approach provides near bare-metal performance.

As a result, tests that require direct access to the device will fail
when run within a guest domain. However, running load-generation and
other tests that do not require direct access to the device and/or
driver within Linux and Windows guest domains is very valuable as such
tests represent how the majority of load will be processed in actual
XenServer installations.

When running tests in guest domains, ensure that you do so with the
XenServer synthetic drivers installed in the guest domain. Installation
of the synthetic drivers is a manual process for some guests. See
XenServer Help for more details.

## Tests that require an integrated build

One of the primary goals of the DDK is to allow partners to create,
compile, and test their drivers with XenServer without requiring a
“back-and-forth” of components with the XenServer engineering team.

However, some tests will only be possible after the driver RPMs and any
accompanying binary RPMs have been supplied to Citrix and integrated
into the XenServer product. Two examples are installing to, and booting
from, Fibre Channel and iSCSI LUNs.

In these cases additional coordination is required after the components
have been provided to Citrix to provide a pre-release XenServer build
with the integrated components for testing.

## Certification & support

### Drivers

Citrix maintains a XenServer Hardware Compatibility List (HCL), found at
[hcl.vmd.citrix.com](http://hcl.vmd.citrix.com). This lists all devices
that have been tested and confirmed to function correctly with the
XenServer product.

In order to be listed on the XenServer HCL, hardware vendors must
utilize the appropriate certification kit, obtainable from
<http://www.citrix.com/ready/hcl>. The test kits contain a mix of manual
and automated tests that are run on a host that contains the hardware to
be certified. In general, two such hosts are required to perform the
tests. Test results are submitted to Citrix for validation, and if they
are approved, the device is listed on the HCL within a small number of
working days, along with a link to the supplemental pack that contains
any necessary driver, if this has not yet been incorporated into the
XenServer product. In general, such supplemental packs will be hosted on
partner web sites, though Citrix may additionally opt to link to (or
host) the pack on its own Knowledge Base site.

For certification of converged or hybrid devices, such as CNAs, *each*
function of the device must be separately certified. This implies that
for a device with both networking and storage (HBA) functionality, both
the networking certification tests and the storage certification tests
must be carried out.

There is no restriction on who is permitted to submit certifications to
the XenServer HCL, for example, it is *not* the case that only the
hardware vendor can submit certifications for their products. Having
said this, Citrix strongly prefers hardware vendors to perform
certification testing, as they are best placed to test all of their
products' features.

Once a device is listed on the HCL, Citrix will take support calls from
customers who are using that device with XenServer. It is expected that
partners who submit devices for inclusion in the HCL will collaborate
with Citrix to provide a fix for any issue that is later found with
XenServer which is caused by said device.

### Userspace software

At present, supplemental packs that contain userspace software to be
installed into dom0 may only be issued by partners who have agreements
in place with Citrix where the partner provides level 1 and level 2
support to their customers.

The reason for this is because Citrix will not necessarily have had the
opportunity to test a supplemental pack of a partner, and hence must
rely on partner testing of the pack as installed on XenServer.
Therefore, only partners who perform testing that has been agreed as
sufficient by Citrix can ship supplemental packs. If a customer installs
a pack that is not from an approved partner, their configuration will be
deemed unsupported by Citrix: any issues found will need to be
reproduced on a standard installation of XenServer, without the pack
installed, if support is to be given.

Partners who wish to produce supplemental packs that contain more than
solely drivers should discuss this with their Citrix relationship
manager as early as possible, in order to discuss what software is
appropriate for inclusion, and what testing should be performed.