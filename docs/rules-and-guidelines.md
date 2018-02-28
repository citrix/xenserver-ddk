# Rules and guidelines

## Kernel modules

Kernel modules must be built and packaged according to the following:

-  Modules must be placed in an RPM.

-  All modules must be located under the directory
    `/lib/modules/<kernel>/updates` where &lt;*kernel*&gt; is the
    version of the kernel.

To ensure a pack is fully conformant, Citrix recommends basing it on one
of the examples in the DDK.

> **Note**
>
> The XenServer build into which a kernel module (driver) is installed *must* be the identical build to the DDK that was used to build the pack in which the driver is contained.
>If it is not, the resulting driver disk will not install on XenServer.

## Post-install scripts

Provided they comply with the following constraints, RPMs may contain
scripts that are invoked during installation. Such scripts might be
necessary in order to add appropriate firewall rules, or log rotation
configuration, that is specific to the pack.

1.  Scripts must not start processes.

1.  Scripts must not assume that XenServer is booted and running (as it
    may be that the pack is being installed as part of an initial
    XenServer installation.

1.  In the light of the previous point, if firewall rules are to be
    added using a post-install script, the script *must* execute
    `iptables-restore < /etc/sysconfig/iptables` *before*
    adding its own rules (using `iptables -A`), and then
    execute `iptables-save > /etc/sysconfig/iptables`. This
    ensures that the default rules are loaded before the collection of
    existing and new rules are saved. Failure to save the existing rules
    will mean that when a pack is installed as part of a XenServer host
    installation, the default XenServer firewall rules will be lost.

If an RPM needs to distinguish between a running host and an
installation environment, the following code fragment may be used:

    if runlevel >/dev/null 2>&1; then
    # running host
    else
    # installation
    fi

The following types of file *must* be placed in the appropriate
directories:

-  Udev rules: must be located in `/etc/udev/rules.d/`.

-  Firmware: must be located in `/lib/firmware/updates/<kernel>`.

-  Configuration details: must be located in `/etc/`.

-  Documentation: must be located in `/usr/share/doc/`

-  Firewall rules: must be added using `iptables -A`, having *first*
    executed
    `iptables-save > /etc/sysconfig/iptables`
    (see above).

## Handling upgrades

On upgrade, pack authors may wish to transfer configuration or state information from the previous installation: this section describes how such a transfer may be achieved.

### Pack upgrade during a XenServer upgrade

When a XenServer host is upgraded, the installer replaces the file system before supplemental packs are installed.
This affects the way individual RPMs interact with upgrades.
To enable configuration files (or other configuration data, such as databases) to be carried over, the installer makes the file system of the previous installation (which is automatically backed-up to another partition) available to the RPM scripts.
This is done through the `XS_PREVIOUS_INSTALLATION` environment variable.

Therefore, in order to migrate state across upgrades, supplemental pack
authors must create a suitable script that runs as part of an RPM
installation, and migrates the state. Specifically, the migration should
be from the old root file system pointed to by
`XS_PREVIOUS_INSTALLATION` to the new file system mounted on `/`.

For an example of how this can be done, see the `%post` script in
`examples/userspace/helloworld-user.spec`.

### Pack upgrade on an existing XenServer installation

It is expected that on each release of XenServer, supplemental pack
authors are likely to release new versions of their packs. However, if a
pack author releases an update between XenServer releases, existing
installations of the old version of the pack would need to be upgraded.
This upgrade path is the responsibility of the pack author, as the
location of the configuration data of the old version is on the root
file system, wherever the RPMs installed it to. The update installation
process upgrades all RPMs contained within the pack, hence these RPMs
should be aware of how to deal with the existence of any relevant
configuration files.

## Uninstallation

Supplemental Packs that only contain userspace packages may include a
script that removes the pack from a host.

-  Uninstalls the packages

-  Uninstalls other packages used to apply the pack

-  Causes xapi to remove the pack from the database

## Building packs in existing build environments

Citrix recognises that many partners have existing build systems that
are used to produce the software that might be integrated into a
supplemental pack. To facilitate this, pack authors are *not* required
to make use of the Driver Development Kit VM, *if* they are producing
packs that do *not* contain drivers (as these need to be compiled for
the correct XenServer kernels).

If a pack author wishes to distribute drivers as part of a supplemental
pack, (or a pack consisting solely of drivers, commonly known as a
Driver Disk), then the driver(s) will need to be compiled using the DDK.
However, there is no barrier to pack authors including the driver disks
that are output by the DDK in their own build processes. Citrix does not
support the compilation of drivers for XenServer in any way other than
using the DDK VM.

To build a supplemental pack (but not a driver) as part of an existing
build process, the only package that is necessary from the Binary
Packages ISO is `update-package`.

These can be used in any environment to create an appropriate pack.
Note, however, that this environment must contain various tools that are
normally found in standard Linux distributions, including `tar`,
`mkisofs`, `sed`, and `rpm`.

> **Warning**
>
> When integrating these scripts as part of another build system, pack authors should bear in mind that Citrix may update these scripts as new versions of XenServer are released.
> Pack authors should ensure that they update this package from the new version of the DDK before building packs for the new version of XenServer.

For pack authors who wish to produce a supplemental pack as an output of
another build system, but who wish to include drivers, the following
procedure should be followed:

1.  Copy the driver source into a running DDK VM the corresponds to the
    build of XenServer that the drivers will be targeted at.

1.  Produce driver *RPMs* (rather than a supplemental pack ISO). This
    can be achieved using the `$(RPM-FILE)` rule in the standard
    `Makefile` provided in the example packs.

1.  By default, this command will output three driver RPMs into
    `/root/rpmbuild/RPMS/x86-64`. Pack authors should ignore the
    `debuginfo` RPM, and take the other RPMs for inclusion in their
    supplemental pack. These RPMs can be treated in the same way as any
    other RPM to be shipped in the pack.

1.  These RPMs should be provided to the non-DDK build system, for
    inclusion in the finished pack. Evidently, this process assumes that
    pack authors will be releasing new versions of their packs more
    frequently than the XenServer kernel is changed, or that introducing
    new versions of RPMs into the alternative build system is more
    acceptable than introducing the DDK as the build system.

## Packaging driver firmware

It is increasingly common for hardware manufacturers to produce
components that load up-to-date firmware from the operating system that
is running on the host machine. XenServer supports this mechanism, and
firmware packages should be installed to
`/lib/firmware/updates/<kernel>`. The firmware should be packaged in an
RPM, and included as part of a supplemental pack.

## Versioning

### Supplemental pack versioning

Authors of supplemental packs are free to use whatever version numbering
scheme they feel is appropriate for their pack. Given that many packs
are likely to include RPMs of existing software, it is suggested that
the pack version number correspond to the version of the software it
contains. For example, if an existing management console RPM is at
version 5, it is likely to be less confusing if the first version of the
supplemental pack that contains this RPM is also version 5.

There is no reason why, if it is simple enough, a pack should not be
suitable for multiple releases of XenServer *provided* that it does not
depend on particular versions of other tools. In practice, Citrix is of
the opinion that packs should be re-released for each new version of
XenServer, in order that they are also fully tested by the authors on
that new release. If a pack author chooses to release one pack for
multiple XenServer releases, they should ensure that the dependency
information expressed in the metadata of the pack uses the `ge`
comparator for the XenServer product, where the version to be compared
against is the lowest supported version of XenServer.

### Kernel module versioning

Each kernel module is built against a specific kernel version. This
kernel version is included in the RPM name to enable multiple instances
to be installed. The version fields of a kernel module RPM are used to
track changes to a driver for a single kernel version. Each time a
driver is released for a particular kernel, the RPM version must be
increased (as would be expected, given that there will have been changes
made to the driver sources).

For example:

    helloworld-modules-xen-2.6.18-128.1.6.el5.xs5.5.0.502.1014-1.0-1.i386.rpm
    driver name = helloworld
    kernel flavour = xen
    kernel version = 2.6.18-128.1.6.el5.xs5.5.0.502.1014
    RPM version = 1.0
    RPM build = 1

| XenServer Kernel version | Kernel module RPM version | Event                                              |
|--------------------------|---------------------------|----------------------------------------------------|
| 2.6.18.128.1.5...        | 1.0-1                     | Initial release of pack                            |
| 2.6.18.128.1.5...        | 1.1-1                     | Driver bug fix                                     |
| 2.6.27.37...             | 2.0-1                     | New XenServer kernel, and new driver version       |

Therefore, if a supplemental pack contains a driver, it will be
necessary to rebuild that driver for each update and major release of
XenServer. Note that hotfixes do not normally change the kernel version,
and hence the same driver can be used until a XenServer update ("service
pack") is released.

As a general rule, if a pack contains only a single driver, it is
strongly recommended that the version numbering of the pack be the same
as that of the driver.

## Packages compiled by, but not in, XenServer

Some of the packages that are included in the XenServer control domain
are taken directly from the base Linux distribution, whilst others are
modified and re-compiled by Citrix. In some cases, certain source RPMs,
when compiled, result in more than one binary RPM. There exist a variety
of packages where XenServer includes some, but not all, of the resulting
binaries; for example, the `net-snmp` package results in the binary
packages `net-snmp` and `net-snmp-utils`, but `net-snmp-utils` is not
included in dom0.

If a supplemental pack author wishes to include a binary package that
falls into this category, that binary package will need to have the
correct build number for the version of XenServer it is to be installed
upon. Because Citrix re-compiles these packages, their build numbers
will have a XenServer-specific build number extension. Therefore, pack
authors will need to obtain these binary RPMs from Citrix.

To enable this process to be as simple as possible, Citrix produces an
extra ISO (`binpkg.iso`) for each release of XenServer that contains all
the packages that fall into this category. Partners should contact
Citrix to obtain this ISO.

## Requirements for submission of drivers for inclusion in XenServer

Citrix encourages hardware vendors to submit any driver disks released
for XenServer to Citrix in order that the drivers may be incorporated
into the next release of the product. In order to make this process as
simple as possible, vendors are requested to take note of the following
requirements:

1.  Any driver submitted must include its full source code, that is
    available under an open source license compatible with the GNU
    General Public License (GPL).

1.  Any binary firmware submitted must either be already publicly
    available under a license allowing re-distribution, or the vendor
    must have a current re-distribution agreement with Citrix.

1.  If a new, (rather than an update to an existing) driver is being
    submitted, Citrix will review it in order to confirm that it is
    compatible with the current support statements made concerning
    XenServer. It may be that a driver is rejected because it is
    monolithic, or enables a feature which is not currently officially
    supported. This may also be the case with radical changes made to
    drivers that are already in the product. Partners who consider that
    their driver(s) fall into this category should contact Citrix as
    early as possible, in order that both organizations' engineering
    teams are able to ascertain how to proceed.

1.  Strict time limits apply to submissions (see below). Vendors should
    make their drivers available to Citrix as soon as possible, rather
    than waiting until these deadlines, as testing may result in fixes
    being required, which then need to be integrated into the product.

1.  Certain drivers are deemed critical to automated testing by Citrix
    of XenServer. Any (entire-driver) updates to these drivers must be
    provided a minimum of 6 weeks prior to the beta RTM date of the
    release in which they are to be included. Partners whose drivers are
    on this list will be informed of this constraint.

1.  All other entire-driver updates (or new drivers) must be provided to
    Citrix a minimum of 5 weeks prior to beta RTM.

1.  Any updates to drivers (for example, no new drivers) received after
    these dates must be in the form of small, targeted fixes for
    specific issues. Patches must be submitted that can be understood by
    a reasonably experienced person, together with a description of the
    flaw that particular patch addresses. Each fix should be in the form
    of a separate patch, with an indication of what the flaw fixed is,
    the effects the flaw would have if it is not addressed, and whether
    such issues have already been seen by customers. Significant
    additions of functionality, or very large patches will not be
    accepted.

1.  Close to the RTM date of the beta of the release concerned, it is
    unlikely that patches will be inserted into the beta release (though
    they may be incorporated into the final release, if they are judged
    to be of sufficient importance). Only in exceptional circumstances
    will patches received fewer than 2 weeks prior to beta RTM be
    incorporated into the beta. The preferred target for all driver
    updates is the beta release, in order to achieve maximum testing
    benefit.

1.  Submitting a GPG Key: Once the key has been generated, when creating
    the update pack, the name of the GPG key is baked into the Yum repo
    metadata. This means the public key file cannot be renamed without
    resigning the Update package.

    When the key-pair has been generated, export the public part (using
    ASCI Armor) and create a ticket on [Citrix Issue
    Tracker](http://tracker.citrix.com/) to include it in the inbox.

## Does XenServer already include a driver for my device?

XenServer includes a wide variety of drivers, including many that are
distributed (inbox) with the kernel that dom0 is based upon. It may
therefore be the case that XenServer includes a driver that enables a
device that is not present on the XenServer Hardware Compatibility List
(HCL). This is particularly the case where a device is sold by multiple
companies, each of which refers to it with a different name.

Because each driver included in XenServer includes information
concerning which PCI device IDs it claims, the simplest way to ascertain
whether a device is supported is to first find its device ID.

If the device is present in a running Linux-based system, the `lspci -v`
command can be used, which will provide output which includes the
information on all devices present in the host. If the `-n` switch is
given, numeric IDs will be provided.

If only the name of the device is known, use the PCI ID database
(<http://pciids.sourceforge.net/pci.ids>) to ascertain what the ID of
the device is. This database will also provide alternate names for the
device, which may of use if the exact name is not listed in the
XenServer HCL.

If the an alternate name for the device is not found on the HCL, then
either the device has not been tested on XenServer, or a driver for it
is not included in XenServer. To confirm whether a suitable driver is
included, consult the list of PCI IDs the XenServer kernel supports,
found in `/lib/modules/<version>/modules.pcimap`.