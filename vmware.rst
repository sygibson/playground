
.. Copyright (c) 2017 RackN Inc.
.. Licensed under the Apache License, Version 2.0 (the "License");
.. Digital Rebar Provision documentation under Digital Rebar master license
.. index::
  pair: vmware; Content Packages

.. _rs_cp_vmware:


vmware
~~~~~~

The following documentation is for vmware content package at version v2.7.0-6-04bccd32b1fc556be12e5b07efbe910237e6b255.

VMware vSphere Tools
====================

A collection of tools for managing VMware vSphere ESXi nodes via Digital
Rebar Provision (DRP) workflows and content.

Kickstart Templating Usage
--------------------------

There are two primary Kickstart templates used for ESXi installs with this
content pack:

  ::

     esxi-install.ks.tmpl      --> Python 2.x based
     esxi-install.py3.ks.tmpl  --> Python 3.x based

Since there is no DRP "runner" (agent) that can run in an ESXi environment,
we can not execute workflow steps post-reboot of the installer.  This causes
a problem since we need to notify DRP that the install has completed and
that we can mark the machine done, and boot to "local" disk.

A small Python snippet is used to make the API call back to DRP to notify that
the install has finished.  But, we have to use the right version of Python,
since ... Python.  Currently this is done in the "%pre" (Python interpreter)
phase.

Adding Custom Kickstart Sections
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ESXi kickstarts support three additional sections (or phases) that extra
actions can be taken:

  1) Pre Stage (%pre): Specifies a script to run before the kickstart configuration is evaluated. For example, you can use it to generate files for the kickstart file to include.
  2) Actual ESXi Installation Stage, uses Kickstart for automation
  3) Post Stage (%post): Runs the specified script after package installation is complete. If you specify multiple %post sections, they run in the order that they appear in the installation script.
  4) Firstboot Stage (%firstboot): Creates an init script that runs only during the first boot. The script has no effect on subsequent boots. If multiple %firstboot sections are specified, they run in the order that they appear in the kickstart file.

DRP's "vmware" content allows you to inject custom %pre, %post, and %firstboot scripts
in to your installation kickstart.  It is up to the operator to understand the ordering
and proper usage of these scripts.

To do so, simply create Templates with your Kickstart snippets defined as you desire.
Then use the "esxi/kickstart-custom-configs" Param structure to include the appropriate
scripts in your kickstart.  NOTE:  the "%pre", "%post", and "%firstboot" separators will
be injected into the Kickstart file for you (do NOT include them in the template).

Licensing ESXI During Install
-----------------------------

Use the "esxi/license" Parameter to set a License to be used during installation
to enable licensed use of ESXi.






params
------

The content package provides the following params.



esxi/skip-tools
===============

If set to "True", the 'tools.t00' module will be removed from
the modules list.  Since this module is over 155 MB in size,
this increases the speed of the install significantly.

This is typically used in a dev/test environment where rapid
(re)build of ESXi hypervisors is occuring.

NOTE: that removing the tools may have an impact on installing
the tools in the Guest VM instances.




vcf-builder/vSwitch
===================

Sets the management vSwitch information to use for
managing the ESXi instance via vCF Builder bringup.




vmware/esxi-hcl-completed
=========================

-| Set to "true" if the Compatibility Checker has already been run for the given host.



esxi/license
============

If specified, sets the license ID to assign to the installed ESXi
VMware vSphere hypervisor.

If not specified, then the hypervisor will be installed in evaluation
mode.




esxi/network-ipaddr
===================

Use this param with "esxi/network-type" set to "manual" to specify
the ESXi nodes network configuration post-kickstart.  NOTE, you must
verify that the IP addressing and Layer 2 characteristics are correct
for proper network connectivity post-reboot.




esxi/serial-console
===================

Sets the serial console for the ESXi host.  By default this is not enabled.

For Packet environment, insure to set com port to "com2", like:

  gdbPort=none logPort=none tty2Port=com2




vcf-builder/cluster-profile
===========================

This parameter is used to identify the Profile name to record
the vCloud Foundation Builder build information.




vcf-builder/serverId
====================

The ESXi server ID string in the cluster.  By default the DRP
Machine UUID will be used as the serverId.

If you choose to set a different value than this default behavior,
then you will have to apply it as an individual Param (or as a
different Profile from the "vcf-builder/cluster-profile" defined
profile) on the given Machine.

You can NOT set the value in the "vcf-builder/cluster-profile" as
that profile is set on every machine in the cluster.




esxi/network-gateway
====================

Use this param with "esxi/network-type" set to "manual" to specify
the ESXi nodes network configuration post-kickstart.  NOTE, you must
verify that the IP addressing and Layer 2 characteristics are correct
for proper network connectivity post-reboot.




esxi/network-netmask
====================

Use this param with "esxi/network-type" set to "manual" to specify
the ESXi nodes network configuration post-kickstart.  NOTE, you must
verify that the IP addressing and Layer 2 characteristics are correct
for proper network connectivity post-reboot.




esxi/shell-remote
=================

If set to "True", enable the remote ESXi shell (SSH access) during the
installation of the VMware vSphere ESXi hypervisor.

By default, the remote shell will not be enabled.




vcf-builder/esxiCredentials-username
====================================

Used to connect to the ESXi instance for vCloud Builder to
manage the node.




vmware/vsan-hcl-completed
=========================

Defines if the machine has run the VMware vSphere VSAN HCL
compliance check tool.




vmware/vsan-hcl-validated
=========================

Defines if the machine has been validated for VMware vSphere VSAN
installation by the validation tool.




vsan/license
============

If specified, sets the license ID to assign to the installed ESXi
VSAN appliance.

If not specified, then VSAN will be installed in evaluation mode.




esxi/kickstart-custom-configs
=============================

This Param allows an operator to create a list of additional Templates
to include during the ESXi kickstart installation phase.  Kicstarts support
three additional installation phases supported by this Param

  %pre   %post   %firstboot

In addition, each phase can use "busybox" (shell) or "python" interpreter
to implement customizations. This results in the "phase" types that you
can set as follow

  pre-busybox       pre-python       post-busybox     post-python
  fistboot-busybox  fistboot-python

Each "phase" has a an array of various templates that can be injected in
that given phase to build up the Kickstart file.  Below is an example

  # YAML example
  - "pre-busybox"
    - "my-pre-busybox-chunk1.ks.tmpl"
    - "my-pre-busybox-chunk2.ks.tmpl"
  - "post-python"
    - "my-post-python3-chunk2.ks.tmpl"
  - "firstboot-busybox"
    - "my-fistboot-busybox-chunk1.ks.tmpl"

  # JSON example
  {
    "firstboot-busybox": [
      "my-fistboot-busybox-chunk1.ks.tmpl"
    ],
    "post-python": [
      "my-post-python3-chunk2.ks.tmpl"
    ],
    "pre-busybox": [
      "my-pre-busybox-chunk1.ks.tmpl",
      "my-pre-busybox-chunk2.ks.tmpl"
    ]
  }

WARNING - for "python" interpreter template types, you must insure you use
          the correct python2 or python3 code for the given version of ESXi
          you are kickstart installing.




esxi/network-hostname
=====================

Use this param with "esxi/network-type" set to "manual" to specify
the ESXi nodes network configuration post-kickstart.  NOTE, you must
verify that the IP addressing and Layer 2 characteristics are correct
for proper network connectivity post-reboot.




vcf-builder/esxiCredentials-password
====================================

Defines the ESXi password for vCF Builder to be able
to access and managed the instance.




vcf-builder/association
=======================

Defines the grouping for the vCloud Foundation Builder association
between elements.

Example:
  sfo01-m01-dc




vmware/esxi-hcl-location
========================

-| The python3 path location to the installed ESXi Compatibility Checker. This should be correctly setup by the "vmware-esxi-hcl-install" Stage.
Note that the Compatibility Checker tool currently requires access to a vCenter server and ESXi already installed on a host to gather the compatibility report information for.  Additionaly, you must have internet access to the VMware API Gateway located at
https://apigw.vmware.com/m4/compatibility/v1



vmware/esxi-hcl-validated
=========================

-| Once the VMware ESXi Compatibility Checker has been completed, this Param will contain the results of the check.  If the machine is marked "compatible" by the VMware tools, then the value will be set to "true", otherwise, it's "false".
If "true" then it is implied the machine is compliant with the HCL.
Note that both this Param and "vmware/esxi-hcl-completed" should be set to "true" to insure that the tooling ran, and the machine passed validation.
However, in DevTest situations, you may set the value to "true" to allow for subsequent installers to complete successfully, regardless of the actual HCL compatibility check results.



vmware/esxi-version
===================

-| This Param can be used to specify which supported version of VMware vSphere to install on a given Machine.  Note that the version must be a supported BootEnv type on the DRP Endpoint.
Supported versions are esxi-550u3b esxi-6u2 esxi-650a esxi-650u2 esxi-670 esxi-670u1
If nothing is specified, the default value is "esxi-670"



esxi/network-dns
================

Use this param with "esxi/network-type" set to "manual" to specify
the ESXi nodes network configuration post-kickstart.  NOTE, you must
verify that the IP addressing and Layer 2 characteristics are correct
for proper network connectivity post-reboot.

Up to two DNS nameservers may be specified by separating them with a
comma (,) without any spaces (eg. "1.1.1.1,1.0.0.1").




esxi/network-type
=================

This Param specifies how the ESXi node should be configured for it's
network interface post-kickstart.  The supported mode types are

  dhcp     (default) requests IP info via DHCP service
  convert  converts the existing DHCP lease to a static assignment
  manual   operator provides values via additonal params

If set to "manual", use the following Params to configure the network

  esxi/network-ipaddr     sets the IP Address
  esxi/network-netmask    sets the subnet Netmask
  esxi/network-gateway    sets the Default Gateway
  esxi/network-dns        sets DNS server(s) (comma, no spaces separated)
  esxi/network-hostname   defines nodes Hostname

All values must be specified for "manual" type.




esxi/shell-local
================

If set to "True", enable the local ESXi shell during the installation
of the VMware vSphere ESXi host.

By default, the shell will not be enabled.








stages
------

The content package provides the following stages.



esxi-fujitsu-vmvisor-installer-6.7-10-install
=============================================

Provides custom VMware BootEnv for Fujitsu specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-FUJITSU&productId=742




esxi-lenovo_esxi6.7u1-10302608_201810-install
=============================================

Provides custom VMware BootEnv for Lenovo specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-LENOVO&productId=742




vcf-builder-deploy
==================

-| This Stage will set the JSON structures needed for an ESXi node.
Requires "ovftool" - which can be acquired from the docker container sygibson/vmtools - which also contains a DRP Runner for running arbitrary workflow.
Param                                 Default vcf-builder/association               sfo01-m01-dc vcf-builder/esxiCredentials-username  root vcf-builder/esxiCredentials-password  RocketSkates vcf-builder/vSwitch                   vswitch0 vcf-builder/serverId                  Machine.UUID



vmware-esxi-install
===================

-| This stage will install VMware vSphere ESXi on the machine of the ESXi and VSAN compatibility checks have succeeded (both set to true).
Future implementations of this should allow for a "force-install" option to allow installation even if the compatibility checks have failed.



esxi-6.7.1-10302608-nec-6.702-install
=====================================

Provides custom VMware BootEnv for NEC specific hardware and drivers
(NOT FOR NEC Express5800/R120h and T120h Series systems)
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




esxi-vmware-esxi-6.7.0-10302608-custom-cisco-install
====================================================

Provides custom VMware BootEnv for Cisco hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-CISCO&productId=742




hpe-esxi-6.7.0-update1-isogen9p-install
=======================================

Provides custom VMware BootEnv for HPE specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HPE&productId=742




vmware-esxi-hcl-validate
========================

-| This task will run the VMware ESXi HCL compatibility checker and determine if the Machine meets the HCL compatibility for ESXi.
see https://labs.vmware.com/flings/esxi-compatibility-checker



esxi-6.7.0-update1-10302608-custom-hitachi_0200_Blade_HA8000-install
====================================================================

Provides custom VMware BootEnv for Hitachi HA8000V Gen10 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




esxi-6.7.0-update1-10302608-custom-hitachi_1200_HA8000VGen10-install
====================================================================

Provides custom VMware BootEnv for Hitachi BladeSymphony and HA8000 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




esxi-dellemc-esxi-6.7u1-10764712-a04-install
============================================

Provides custom VMware BootEnv for Dell EMC specific hardware and drivers
For more details, and Download ISO see:
  https://www.dell.com/support/home/us/en/19/drivers/driversdetails?driverid=6g5t3




vcf-builder-settings
====================

-| This Stage will set the JSON structures needed for an ESXi node to be included in the vCloud Foundation Builder built cluster. It must be run on Sledgehammer, prior to ESXi being installed.
The following Optional Params will be injected in to the JSON esxiHostSpec stanzas if set.  If not specified, the default values will be used.
Param                                 Default vcf-builder/association               sfo01-m01-dc vcf-builder/esxiCredentials-username  root vcf-builder/esxiCredentials-password  RocketSkates vcf-builder/vSwitch                   vswitch0 vcf-builder/serverId                  Machine.UUID



vmware-esxi-hcl-install
=======================

-| This stage will install all prerequisites for the VMware HCL compatibility checker, and the python3 tool itself (compchecker.py).
see  https://labs.vmware.com/flings/esxi-compatibility-checker



vmware-vsan-hcl-install
=======================

-| This stage will install all prerequisites for the VMware VSAN HCL compatibility checker tool.
WARNING  this is not completed - the tool does not exist for LINUX yet



esxi-6.7.1-10302608-nec-gen-6.7-install
=======================================

Provides custom VMware BootEnv for NEC Express5800/R120h and T120h Series specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




vmware-vsan-hcl-validate
========================

-| This stage will verify that the Machine is HCL compliant for VSAN installation.







tasks
-----

The content package provides the following tasks.



vmware-esxi-install
===================

-| If the given Machine has passed both ESXi and VSAN HCL compatibility checks, then install the VMware vSphere ESXi version specified.
Uses the "vmware/esxi-version" Param (enum type) to define which supported version to install.



vmware-vsan-hcl-install
=======================

-| This task will install all prerequisites for the VMware VSAN HCL compatibility checker tool.
WARNING  this is not completed - the tool does not exist for LINUX yet



vmware-vsan-hcl-validate
========================

-| This task will verify that the Machine is HCL compliant for VSAN installation.



vcf-builder-deploy
==================

-| This task will use "ovftool" to deploy a vCloud Foundation builder appliance to a specified ESXi node.



vcf-builder-settings
====================

-| This task will set the JSON structures needed for an ESXi node to be included in the vCloud Foundation Builder built cluster. It must be run on Sledgehammer, prior to ESXi being installed.



vmware-esxi-hcl-install
=======================

-| This task will install all prerequisites for the VMware HCL compatibility checker, and the tool itself.
see  https://labs.vmware.com/flings/esxi-compatibility-checker



vmware-esxi-hcl-validate
========================

-| This task will run the VMware ESXi HCL compatibility checker and determine if the Machine meets the HCL compatibility for ESXi.
see  https://labs.vmware.com/flings/esxi-compatibility-checker







workflows
---------

The content package provides the following workflows.



esxi-6.7.1-10302608-nec-6.702-install
=====================================

Provides custom VMware BootEnv for NEC specific hardware and drivers
(NOT FOR NEC Express5800/R120h and T120h Series systems)
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




esxi-fujitsu-vmvisor-installer-6.7-10-install
=============================================

Provides custom VMware BootEnv for Fujitsu specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-FUJITSU&productId=742




esxi-6.7.1-10302608-nec-gen-6.7-install
=======================================

Provides custom VMware BootEnv for NEC Express5800/R120h and T120h Series specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




esxi-6.7.0-update1-10302608-custom-hitachi_0200_Blade_HA8000-install
====================================================================

Provides custom VMware BootEnv for Hitachi HA8000V Gen10 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




esxi-dellemc-esxi-6.7u1-10764712-a04-install
============================================

Provides workflow to install ESXi for Dell EMC specific hardware.
For more details, and Download ISO see:
  https://www.dell.com/support/home/us/en/19/drivers/driversdetails?driverid=6g5t3




esxi-lenovo_esxi6.7u1-10302608_201810-install
=============================================

Provides custom VMware BootEnv for Lenovo specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-LENOVO&productId=742




esxi-vmware-esxi-6.7.0-10302608-custom-cisco-install
====================================================

Provides custom VMware BootEnv for Cisco hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-CISCO&productId=742




esxi-6.7.0-update1-10302608-custom-hitachi_1200_HA8000VGen10-install
====================================================================

Provides custom VMware BootEnv for Hitachi BladeSymphony and HA8000 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




hpe-esxi-6.7.0-update1-isogen9p-install
=======================================

Provides custom VMware BootEnv for HPE specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HPE&productId=742








bootenvs
--------

The content package provides the following bootenvs.



esxi-vmware-esxi-6.7.0-10302608-custom-cisco-install
====================================================

Provides custom VMware BootEnv for Cisco hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-CISCO&productId=742




esxi-6.7.0-update1-10302608-custom-hitachi_0200_Blade_HA8000-install
====================================================================

Provides custom VMware BootEnv for Hitachi HA8000V Gen10 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




esxi-6.7.1-10302608-nec-6.702-install
=====================================

Provides custom VMware BootEnv for NEC specific hardware and drivers
(NOT FOR NEC Express5800/R120h and T120h Series systems)
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




esxi-dellemc-esxi-6.7u1-10764712-a04-install
============================================

Provides custom VMware BootEnv for Dell specific hardware and drivers
For more details, and Download ISO see:
  https://www.dell.com/support/home/us/en/19/drivers/driversdetails?driverid=6g5t3




esxi-lenovo_esxi6.7u1-10302608_201810-install
=============================================

Provides custom VMware BootEnv for Lenovo specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-LENOVO&productId=742




esxi-fujitsu-vmvisor-installer-6.7-10-install
=============================================

Provides custom VMware BootEnv for Fujitsu specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-FUJITSU&productId=742




esxi-6.7.1-10302608-nec-gen-6.7-install
=======================================

Provides custom VMware BootEnv for NEC Express5800/R120h and T120h Series specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM_ESXI67U1_NEC&productId=742




esxi-6.7.0-update1-10302608-custom-hitachi_1200_HA8000VGen10-install
====================================================================

Provides custom VMware BootEnv for Hitachi BladeSymphony and HA8000 specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HITACHI&productId=742




hpe-esxi-6.7.0-update1-isogen9p-install
=======================================

Provides custom VMware BootEnv for HPE specific hardware and drivers
For more details, and Download ISO see:
  https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI67U1-HPE&productId=742







