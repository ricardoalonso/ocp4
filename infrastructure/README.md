# Infrastructure setup

This project is to setup an Openshift Container Platform PoC basic infrastructure and allow an easier and faster deployment.

The playbook `setup_bastion.yaml` should be use to setup the following services (into the bastion machine):

* DNS
* DHCP
* TFTP/PXE
* Apache (for RHCOS image)

##### DNS

It defines all the entries necessary to install and run OCP, as listed [here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.4/html-single/installing_on_bare_metal/index#installation-dns-user-infra_installing-bare-metal). 
It also treats the load balancer as a separate machine, but it can be deployed in the same as the bastion. 
DNS forward and reverse are configured. 
The names of the hosts in the inventory are used to configure the DNS entries.

##### DHCP

Sets up all the entries with fixed ip for the macs, to avoid having to configure manually each member. The hostname will also be look up via DNS. 

##### TFTP/PXE

Configures the pxe for each member, according to it's group. The pxe will boot the each member to install the RHCOS, using the ignition files that must placed into `/var/www/html/configs` on the bastion machines, with their default names.

The playbook also downloads the kernel and initram image for RHCOS and place them on the correct locations. You must use the variables `ocp_major_version` and `rhcos_version` to define the correct version to download. 

##### Apache

Starts the apache on port 8080 to serve the RHCOS image (downloaded by the playbook) and ignition files for the OCP installation. 

#### Requirements for the bastion machine:

1. Must be running RHEL 7 or 8 (Centos was not tested)
1. Must be corrected subscribed to Red Hat or Satellite server (no extra repository needed) 
1. Must have at least one network interface on the subnet used to perform the OCP installation, or the DHCP will fail to start. 

#### How to setup 

After installing the operational system and performed the configuration for user and keys (for ansible), you must edit the file `inventory` to reflect to your setup:

**Do NOT modify the group names!** 
The playbook uses the name in the inventory, combined with the value of `cluster_domain` variable to compose the names for the DNS and DHCP services.

````
[bastions]
bastion host_ip=10.11.12.1 mac_address=52:54:00:ab:cd:01

[lbs]
lb host_ip=10.11.12.9 mac_address=52:54:00:ab:cd:09

[masters]
control01 host_ip=10.11.12.11 mac_address=52:54:00:ab:cd:11
control02 host_ip=10.11.12.12 mac_address=52:54:00:ab:cd:12
control03 host_ip=10.11.12.13 mac_address=52:54:00:ab:cd:13

[nodes]
compute01 host_ip=10.11.12.21 mac_address=52:54:00:ab:cd:21
compute02 host_ip=10.11.12.22 mac_address=52:54:00:ab:cd:22
compute03 host_ip=10.11.12.23 mac_address=52:54:00:ab:cd:23

[bootstraps]
bootstrap host_ip=10.11.12.10 mac_address=52:54:00:ab:cd:10

[all:vars]
ocp_major_version = 4.4
ocp_minor_version = 4.4.4
rhcos_version = 4.4.3
cluster_domain = mycluster.local
# the wildcard domain will be composed later: wildcard_domain + cluster_domain
wildcard_domain = *.apps
cluster_subnet = 10.11.12.0
cluster_netmask = 255.255.255.0
cluster_reverse_net = 12.11.10
dhcp_range = 10.11.12.50 10.11.12.99

````

The inventory could be used later on, but at this moment, only this information is necessary to setup the bastion machine. 

When complete, just run ansible-playbook command:

````
$ ansible-playbook setup_bastion.yaml
````

#### TODO

* The loadbalancer setup 
* Generate a shell script to create/start the VM's on Libvirt, based on the inventory. 
* Comments and suggestions are welcomed. :-) 
