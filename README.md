# vbmcctl
Wrapper for the **virtualbmc** (**vbmc**) command that allows for predefined BMC device configuration.

The **virtualbmc** (**vbmc**) command must be installed for this command to function. The **virtualbmc** package is commonly named: ***python-virtualbmc*** 

(In openSUSE, this package can be found in the **Cloud:OpenStack:Master** repository)


## Usage
```
vbmcctl create|delete|reset|list [config_file]
```
The **vbmcctl** command can (should?) be run as a regular user as it uses **sudo** to create the virtual BMC devices.

A list of options and their descriptions:

Option | Description
------------ | -------------
**create** |		create the virtusl BMC devices defined in the config file
**delete** |		delete the virtual BMC devices defined in the config file
**reset** |	reset (delete and then create) the virtual BMC devices defined in the config file
**list** |		list the virtual BMC devices defined in the config file

## Config File
The default config file is: **/etc/vbmcctl.cfg**

If this file does not exist, and a config file is not supplied on the command line, the command will exit with an error.

If the variables in the config file are empty the command will exit with an error.

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
 
	 VIRTUAL_BMC_LIST="controller01,192.168.124.201,623,admin,linux compute01,192.168.124.202,623,admin,linux"

If you want to use the default value for a field just leave that field empty.

***Example 2*** (using default username and password):

	 VIRTUAL_BMC_LIST="controller01,192.168.124.201,623,, compute01,192.168.124.202,623,,"

The VM names should only be VMs that are currenlty defined in Libvirt and cani be seen using the '**virsh list --all**' command.

## Detailed Description
When a virtual BMC device is created a veth pair is created for the BMC device (***VM_NAME*****-bmc** and ***VM_NAME*****-bmc-nic**). The IP address for the BMC device is added to the ***-bmc*** interface and the ***-bmc-nic*** interface is attached to the bridge defined in the **VIRTUAL_BMC_NETWORK** variable in the config file.

```
VM_NAME-bmc-nic <-- VM_NAME-bmc <-- BMC_IP
     |
     V
VIRTUAL_BMC_NETWORK
```
Any machine that can access the VIRTUAL_BMC_NETWORK can use the **ipmitool** command to interact with the virtual BMC device.

A virtual BMC device is then created and associeated with the BMC IP address and the device is started.

### Example *ipmitool* commands:
```
# Power virtual machine on, off, graceful off, NMI and reset
ipmitool -I lanplus -U <BMC_USERNAME> -P <BMC_PASSWORD>  -H <BMC_IP> power on|off|soft|diag|reset

# Check the power status
ipmitool -I lanplus -U <BMC_USERNAME> -P <BMC_PASSWORD> -H <BMC_IP> power status

# Set the boot device to network, hd or cdrom
ipmitool -I lanplus -U <BMC_USERNAME> -P <BMC_PASSWORD>  H <BMC_IP> chassis bootdev pxe|disk|cdrom

# Get the current boot device
ipmitool -I lanplus -U <BMC_USERNAME> -P <BMC_PASSWORD> -H <BMC_IP> chassis bootparam get 5

# Get the current boot device
ipmitool -I lanplus -U <BMC_USERNAME> -P <BMC_PASSWORD> -H <BMC_IP> chassis bootparam get 5
```

The individual BMC devices can be managed using the **vbmc** command. It is important to note that the virtual BMC devices are created as the root user (using sudo) so you must also use the **vbmc** command as root to manage the devices (***sudo vbmc*** ...).

### Example vbmc commands:
```
# List the BMC devices
sudo vbmc list

# Stop a BMC device
sudo vbmc stop <VM_NAME>

# Start a BMC device
sudo vbmc start <VM_NAME>

# Display status of a BMC device
sudo vbmc status <VM_NAME>
```

### Rebooting the host machine:
When the host machine is rebooted the virtual BMC devices will still be defined but will not start because their veth pairs do not exist. The easiest way to fix this is to run the following command to delete and then recreate the virtual BMC devices and thier veth pairs:
```
vbmcctl reset
```
(or if you are using a config file other then the default)
```
vbmcctl reset <CONFIG_FILE>
```

### Troubleshooting:
Sometimes a virtual BMC device will just stop running on its own. You can use the following command to restart the device:
```
sudo vbmc start <VM_NAME>
```
If that doesn't work then just reset all of the BMC devices as described above. Reseting all of the BMC devices is safe to do at any time. If an IPMI command is being issued to the device when it is being reset it will not be received by the device. The command will need to be reissued when the BMC device is up and running again.