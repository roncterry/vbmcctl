# vbmcctl
Wrapper for the virtualbmc (vbmc) command that allows for predefined BMC device configuration

## Usage: 
```
vbmcctl create|delete|reset|list [config_file]
```

## Config File
The default config file is: **/etc/vbmcctl.cfg**

If this file does not exist and a config file is not supplied on the command line the command will exit.

## Config File Syntax

**VIRTUAL_BMC_NETWORK**

This should be the name of one of your virtual networks/bridges. If it is a Libvirt network then it must match the name of the bridge created by the Libvirt network.

***Example:***

	VIRTUAL_BMC_NETWORK="virbr0"
	

**VIRTUAL_BMC_LIST**

The BMC entries in VIRTUAL_BMC_LIST should be a space delimited list of comma delimited values in this order: 

 **VM Name**  (name of the VM in Libvirt)
 **Host BMC IP**  (IP address on the host the BMC listens on: Default=127.0.0.1)
 **BMC Port ** (port the BMC will listed on: Default=623)
** BMC Username**  (username to log in to the BMC as: Default=admin)
** BMC Password**  (password for the BMC user: Default=password)

***Example 1:***
 
	 VIRTUAL_BMC_LIST="controller01,192.168.124.1,623,admin,linux compute01,192.168.124.1,6231,admin,linux"

If you want to use the default value for a field just leave that field empty.

***Example 2*** (using default username and password):

	 VIRTUAL_BMC_LIST="controller01,192.168.124.1,623,, compute01,192.168.124.1,6231,,"

The VM names should only be VMs that are currenlty defined in Libvirt and cani be seen using the '**virsh list --all**' command.
