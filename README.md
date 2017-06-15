## OpenvSwitch compile - RHEL 7, CentOS 7, Fedora 25

*Note:* for Fedora, use `dnf` instead of `yum`

I will not advocate why we should use OpenvSwitch instead of Linux bridges, reasons are numerous. OpenvSwitch brings all Software Defined Network (SDN) capabilities within linux Operating System.
Open vSwitch is a production quality, multilayer virtual switch licensed under the open source Apache 2.0 license.  It is designed to enable massive network automation through programmatic extension, while still supporting standard management interfaces and protocols (e.g. NetFlow, sFlow, IPFIX, RSPAN, CLI, LACP, 802.1ag).  In addition, it is designed to support distribution across multiple physical servers similar to VMware's vNetwork distributed vswitch or Cisco's Nexus 1000V.

To use the latest rpm version of OpenvSwitch (2.7.0 for example) instead of the one's available in the epel repository.

Install necessary binaries to build rpm:

    $ sudo yum -y install gcc openssl-devel selinux-policy-devel

Create rpm build environment:

    $ sudo yum install -y rpmdevtools
    $ cd ~
    $ rpmdev-setuptree
    
The latest command will create the folder  `~/rpmbuild`.
   
Download and extract in `~/rpmbuild/SOURCES` the latest version of OpenvSwitch binary from http://openvswitch.org:

    $ wget -qO- http://openvswitch.org/releases/openvswitch-2.x.y.tar.gz | tar xvz -C ~/rpmbuild/SOURCES
   
Build rpms from openvswitch spec file:

    $ rpmbuild -bb ~/rpmbuild/SOURCES/openvswitch-2.x.y/rhel/openvswitch.spec   

OpenvSwitch rpms can now be installed from `~/rpmbuild/RPMS` folder:

    $ sudo yum -y install ~/rpmbuild/RPMS/noarch/openvswitch-selinux-policy-2.x.y-z.noarch.rpm
    $ sudo yum -y install ~/rpmbuild/RPMS/x86_64/openvswitch-2.x.y-z.x86_64.rpm
    
Enable and start OpenvSwitch:

    # systemctl enable openvswitch
    # systemctl start openvswitch

Run our first OpenvSwitch command:

    # ovs-vsctl show
        ovs_version: "2.7.0"
 
In my example i am running the version *2.7.0* of OpenvSwitch.


 
## Libvirt & OpenvSwitch
We assume that libvirtd and OpenvSwitch are enable and running on the system.
Let's create three (03) subnets with OpenvSwitch bridges (`ovsbr-dmz`, `ovsbr-int`, `ovsbr-noc`) and map them to libvirt networks.
OpenvSwitch subnets are identified by the following parameters:
<pre>
ovsbr-noc:
   ip: 10.30.0.1/24
ovsbr-int:
   ip: 10.10.0.1/24
ovsbr-dmz:
   ip: 10.20.0.1/24 
</pre>

###Permanent network configurations on RedHat, CentOS and Fedora:

`/etc/sysconfig/network-scripts/ifcfg-ovsbr-int`

		DEVICE=ovsbr-int
		ONBOOT=yes
		DEVICETYPE=ovs
		TYPE=OVSBridge
		BOOTPROTO=static
		IPADDR=10.10.0.1
		NETMASK=255.255.255.0
		HOTPLUG=no
		ZONE=public

`/etc/sysconfig/network-scripts/ifcfg-ovsbr-dmz`

		DEVICE=ovsbr-dmz
		ONBOOT=yes
		DEVICETYPE=ovs
		TYPE=OVSBridge
		BOOTPROTO=static
		IPADDR=10.20.0.1
		NETMASK=255.255.255.0
		HOTPLUG=no
		ZONE=public

`/etc/sysconfig/network-scripts/ifcfg-ovsbr-noc`

		DEVICE=ovsbr-noc
		ONBOOT=yes
		DEVICETYPE=ovs
		TYPE=OVSBridge
		BOOTPROTO=static
		IPADDR=10.30.0.1
		NETMASK=255.255.255.0
		HOTPLUG=no
		ZONE=public

These commands on system boot will create OpenvSwitch bridges with the configured ip addresses.

Create three (03) Libvirt network's xml files mapped to the OpenvSwitch bridges:

`ovsbr-dmz.xml`

		<network>
		  <name>ovsbr-dmz</name>
		  <forward mode='bridge'/>
		  <bridge name='ovsbr-dmz'/>
		  <virtualport type='openvswitch'/>
		</network>
		
`ovsbr-int.xml`

		<network>
		  <name>ovsbr-int</name>
		  <forward mode='bridge'/>
		  <bridge name='ovsbr-int'/>
		  <virtualport type='openvswitch'/>
		</network>
		
`ovsbr-noc.xml`		

		<network>
		  <name>ovsbr-noc</name>
		  <forward mode='bridge'/>
		  <bridge name='ovsbr-noc'/>
		  <virtualport type='openvswitch'/>
		</network>

Define and start the network

	  # virsh net-define ovsbr-int.xml
	  # virsh net-autostart ovsbr-int
	  # virsh net-start ovsbr-int

  


## Ansible
