# Additional Resources

In addition to supplemental packs, a variety of mechanisms are available
for partners to interface with XenServer, and add value to the user
experience. This chapter overviews the mechanisms, and provides web
links to further information.

If pack authors have any questions concerning what should (or should
not) be included in a pack, or how a particular customization goal might
be achieved, they are encouraged to contact their Citrix technical
account manager.

## Xen API plug-ins

Whilst the Xen API provides a wide variety of calls to interface with
XenServer, partners have the opportunity to add to the API by means of
XAPI plug-ins. These consist of Python scripts that are installed as
part of supplemental packs, that can be run by using the
`host.call_plugin` XAPI call. These plug-ins can perform arbitrary
operations, including running commands in dom0, and making further XAPI
calls, using the XAPI Python language bindings.

For examples of how XAPI plug-ins can be used, please see the example
plug-ins in the `/etc/xapi.d/plugins/` directory of a standard XenServer
installation.

## XenCenter plug-ins

XenCenter plug-ins provides the facility for partners to add new menus
and tabs to the XenCenter administration GUI. In particular, new tabs
can have an embedded web browser, meaning that existing web-based
management interfaces can easily be displayed. When combined with Xen
API plug-ins to drive new menu items, this feature can be used by
partners to integrate features from their supplemental packs into one
centralized management interface for XenServer.

For further information, please see
<http://community.citrix.com/display/xs/XenCenter+Plugins>.

## XenServer SDK

The XenServer SDK VM is a virtual machine appliance, ready to be
imported into a XenServer host. It contains the complete set of
XenServer SDKs, plus a complete Linux-based development environment.
Language bindings are available for C, C\#, Python, and Java, and a
Windows PowerShell snap-in is also available. The SDK VM allows partners
to easily develop solutions that utilize the rich API exposed by all
XenServer hosts. The SDK VM is intended only as a convenient development
environment: for an application developed against the appropriate
language bindings to be distributed, only the relevant bindings are
required (*not* the entire SDK VM).

For further information, please see
<http://community.citrix.com/display/xs/Download+SDKs>.
