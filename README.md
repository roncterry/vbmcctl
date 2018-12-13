# vbmcctl
Wrapper for the **virtualbmc** (**vbmc**) command that allows for predefined BMC device configuration.

The **virtualbmc** (**vbmc**) command must be installed for this command to function. The **virtualbmc** package is commonly named: ***python-virtualbmc*** or **python3-virtualbmc**

(In openSUSE, this package can be found in the **Cloud:OpenStack:Master** repository)


## Usage
```
vbmcctl create|delete|reset|list|start|stop|restart|status [config=<config_file>] [<vm_name> [<vm_name> ...]] [nocolor]
```
The **vbmcctl** command should be run as a regular user as it uses **sudo** to create the virtual BMC devices. 

An alternate config file can be specifice on the CLI with the **config=** option. If an alternate config file is specifice then the default config file is ignored.

One or more VMs that are listed in the configuration file can be specified on the CLI. If this is the case the the action is only performed on this list of VMs rather then all VMs listed in the config file.

A list of options and their descriptions:

Action | Description
------------ | -------------
**create** |		create the virtusl BMC devices defined in the config file or specified on the CLI
**delete** |		delete the virtual BMC devices defined in the config file or specified on the CLI
**reset** |	reset (delete and then create) the virtual BMC devices defined in the config file or specified on the CLI
**list** |		list the virtual BMC devices defined in the config file or specified on the CLI along with their status
**start** |	start the virtual BMC devices defined in the config file or specified on the CLI
**stop** |	stop the virtual BMC devices defined in the config file or specified on the CLI
**restart** |	stops and then starts the virtual BMC devices defined in the config file or specified on the CLI
**status** |		displays the status of the virtual BMC devices defined in the config file status or specified on the CLI

--

Option | Description
------------ | -------------
**nocolor** |		turns off colorization of the output


## Config File
The default config file is: **/etc/vbmcctl.cfg**

If this file does not exist, and a config file is not supplied on the command line using the **config=** option, the command will exit with an error.

If the variables in the config file are empty the command will exit with an error.

## Config File Syntax

**VIRTUAL_BMC_NETWORK**

This should be the name of one of your virtual networks/bridges. If it is a Libvirt network then it must match the name of the bridge created by the Libvirt network.

***Example:***

	 VIRTUAL_BMC_NETWORK="virbr0"

**VIRTUAL_BMC_LIST**

The BMC entries in VIRTUAL_BMC_LIST should be a space delimited list of comma delimited values in this order: 

Value | Description
------------ | -------------
VM Name |  name of the VM in Libvirt
Host BMC IP |  IP address on the host the BMC listens on: Default=127.0.0.1 
            |  This addres can contain a CIDR mask or not: i.e. 192.168.100.101/24 or 192.168.100.101
            |  If no CIDR mask is specified it will first try to determine the CIDR mask of the VIRTUAL_BMC_NETWORK. If it can't determine one it will use the CIDR mask of /24
BMC Port | port the BMC will listed on: Default=623
 BMC Username |  username to log in to the BMC as: Default=admin
 BMC Password |  password for the BMC user: Default=password
Libvirt URI |  Libvirt URI of the Libvirt server running the VM: Default: qemu:///system

***Example 1:*** (with 2 BMC devices)

	 VIRTUAL_BMC_LIST="controller01,192.168.124.201,623,admin,linux,qemu:///system compute01,192.168.124.202,623,admin,linux,qemu:///system"

If you want to use the default value for a field just leave that field empty.

***Example 2*** (with 2 BMC devices using default username and password):

	 VIRTUAL_BMC_LIST="controller01,192.168.124.201,623,,, compute01,192.168.124.202,623,,,"

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

### Example *vbmc* commands:
```
# List the BMC devices
sudo vbmc list

# Stop a BMC device
sudo vbmc stop <VM_NAME>
(same as vbmcctl stop <VM_NAME>)

# Start a BMC device
sudo vbmc start <VM_NAME>
(same as vbmcctl start <VM_NAME>

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
vbmcctl reset config=<CONFIG_FILE>
```

### Troubleshooting:
Sometimes a virtual BMC device will just stop running on its own. You can use the following command to restart the device:
```
sudo vbmc start <VM_NAME>
```
or
```
vbmcctl start <VM_NAME>
```
If that doesn't work then just **reset** all of the BMC devices as described above. Reseting all of the BMC devices is safe to do at any time. If an IPMI command is being issued to the device when it is being reset it will not be received by the device. The command will need to be reissued when the BMC device is up and running again.