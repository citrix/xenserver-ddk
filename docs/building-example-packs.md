# Building the example packs

To make the process of building an Update package as easy as possible, we have included a number of examples in the Driver Development Kit (DDK), which Citrix makes available to our partners with each build.

Import the DDK onto the XenServer host.
Refer to the instructions described in `/root/examples/README.txt`, which outlines the example scripts for generating GPG test keys as well as the various configuration options for generating a pack.

For most cases, copying the example build files and customizing them with pack specific information should be sufficient.

**Pack UUIDs**: Each pack will have a UUID included in the metadata, which is required to build an Update package.
The UUID must be unique to the Update being generating:

-  The same UUID should not refer to different Updates.

-  The same Update should not have multiple UUIDs.

A number of examples are supplied in the DDK under `/root/examples`.
These include:

-  **Userspace**: a simple example of a pack containing only programs and files that relate to a userspace.

-  **Driver**: a simple kernel driver.

-  **Combined**: an example which contains kernel and userspace files.

There are specific rules for packaging kernel device drivers.
For more information, see [Rules and guidelines](./rules-and-guidelines.md).

Within each directory there is a:

-  **Source tree**: a directory containing a collection of files.

-  **Specification file**: a file that describes how to build an RPM.

-  **Makefile**: a file used to automate the creation of a supplemental pack.

To build a specific example, use the following commands:

    cd /root/example/<dir>
    make 

This will result in the following files being created:

-  **&lt;*pack*&gt;.iso** - the supplemental pack CD image.

Where &lt;*pack*&gt; is the name of the pack.

## Test Signing Key Generation

When the first example pack is built, a new GnuPG key pair is created.
You will be prompted for a passphrase that must be entered whenever the
private key is used to sign a pack. This key pair is only intended for
developer testing. See --- for information on generating the key pair to
use for released Supplemental packs.

To allow partners to release software that is installable in Dom0,
Citrix requires partners provide us with the public key corresponding to
their GPG key pair generated with the following requirements:

-  ASD Type = RSA

-  Bits = 2048

-  Expiry date = preferably none, else &gt;10 years.

-  No support for subkeys as RPM does not handle this properly.

-  Naming convention: RPM-GPG-KEY-&lt;VENDOR&gt;

## Installing the Test Key

Before a pack that has been signed with a test key can be installed on
any XenServer hosts, the public key must be imported into the host on
which the pack should be installed.

1.  Copy the public key from the DDK VM to the host using the following
    command:

         DDK# scp /root/RPM-GPG-KEY-DDK-Test root@XenServer:

1.  Import the key on the host using the following command:

         XS# /opt/xensource/debug/import-update-key RPM-GPG-KEY-DDK-Test

> Note
>
> To allow developer testing, a script is now included in Dom0 that enables our partners to import an update key manually:
>
>```bash
>/opt/xensource/debug/import-update-key <PATH-TO-KEY-FILE>
>```