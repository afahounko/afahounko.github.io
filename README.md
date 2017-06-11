## afahounko.github.io

## OpenvSwitch compile - RHEL7, CentOS 7, Fedora 25

**Note: for Fedora, use `dnf` instead of `yum`

I will not advocate why we should use OpenvSwitch instead of Linux bridges, reasons are numerous. OpenvSwitch brings all Software Defined Network (SDN) capabilities within linux Operating System.
Open vSwitch is a production quality, multilayer virtual switch licensed under the open source Apache 2.0 license.  It is designed to enable massive network automation through programmatic extension, while still supporting standard management interfaces and protocols (e.g. NetFlow, sFlow, IPFIX, RSPAN, CLI, LACP, 802.1ag).  In addition, it is designed to support distribution across multiple physical servers similar to VMware's vNetwork distributed vswitch or Cisco's Nexus 1000V

To use the latest rpm version of OpenvSwitch (2.7.0 for example) instead of the one's available in the epel repository.

Create rpm build environment:

    $ sudo yum install -y rpmdevtools
    $ cd ~
    $ rpmdev-setuptree
    
The latest command will create  `rpmbuild` folder.
   
Download the latest version of OpenvSwitch binary from http://openvswitch.org:

    $ wget -qO- http://openvswitch.org/releases/openvswitch-2.x.y.tar.gz | tar xvz -C ~/rpmbuild/SOURCES

Install necessary binaries:

    $ sudo yum -y install gcc openssl-devel selinux-policy-devel
   
Build rpm from spec file:

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
 
** In my example i am ruuning the version 2.7.0 of OpenvSwitch.


 
## Libvirt & OpenvSwitch

## Ansible
