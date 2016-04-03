####
Integration of Open Stack with Open Daylight
####

Welcome to Open Stack Inteagration with Open Daylight Manual ! 

This document is based on `the official Documentation <https://wiki.opendaylight.org/view/OpenStack_and_OpenDaylight>`_

:Version: 1.0
:Authors: Vishal Sharma
:Keywords: OpenStack, Opendaylight, Integration of Openstack with opendaylight

===============================

**Author:**

Copyright (C) `Vishal Sharma <https://ca.linkedin.com/in/vishalsharma12>`_

================================

.. contents::

Installation of Open Stack
==========================

In this document, I can`t specify the installation of Open Stack. I have followed the `OpenStack docs <http://docs.openstack.org/kilo/install-guide/install/apt/content/>`_

Installation of Open Daylight
=============================

**Prerequisites** :

1. Open Daylight software runs on platform independent Java virtual machine and can be installed on any Operating system on which Java is installed.
2. The basic requirement is to install it on a system with a multi-core processor and 8 GB RAM in order to get optimum results.

* Command to install java jdk on Linux (Ubuntu)::

    #apt-get install openjdk-7-jdk

Download the latest version of Open Daylight from their `official website <https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.4.0-Beryllium/distribution-karaf-0.4.0-Beryllium.tar.gz>`_ by using wget command.
 
* Uncompress the packages::

    #tar -xvf distribution-karaf-0.4.0-Beryllium.tar.gz

* To use the OpenFlow version 1.3, make the changes in the file **etc/custom.properties** in line number 82. Make sure this line is uncommented::

    ovsdb.of.version=1.3
 
* Start the controller::

    ./bin/start
Controller will start subsequently, in case ports have to be checked, **netstat –ntl** is the command.

* To take the access of controller as a client::

    ./bin/client –u karaf

.. Image:: https://github.com/sharma93vishal/Openstack-Opendaylight/blob/master/Images/Picture1.png

* To use Open Daylight with Open Stack, the features of Open Stack have to be installed on the controller::

    opendaylight-user@root>feature:install odl-ovsdb-openstack odl-dlux-all
    
    Command given above will install all the features to connect with Open Stack.

* After this, to check whether everything works fine, use curl command. It will show empty network list::

    curl -u admin:admin http://10.0.0.100:8181/controller/nb/v2/neutron/networks

Erase all VMs, Networks, Routers & Ports in Controller Node 
===========================================================

In order to allow Open Daylight to work with Open Stack, all the Virtual Machines, Routers, Networks, Subnets and ports are needed to be removed.
Following steps will guide you through the cleaning process.

* Delete all the instances::

    # nova delete <instance names>
* Remove all the routers::

    # neutron router-delete <router name>
* Delete all the subnets & networks::

    # neutron subnet-delete <subnet name>
    # neutron net-delete <net name>
* Check that all ports have been cleaned – at this point, this should be an empty list::

    # neutron port-list
* Stop the neutron-server::

    To avoid the conflict between Neutron and Open Daylight, neutron-server has to be shutdown.

    # service neutron-server stop

Open vSwitches in Compute & Network node 
========================================
The Neutron OVS plugin has to be deleted from compute & Network node because Neutron is not handling OVS switches no more. So all the configurations of the OVS switches are needed to be cleaned.

* Delete the neutron ovs-plugin agent::

    # apt-get purge neutron-plugin-openvswitch-agent
* Stop the OVS switches::

    # service openvswitch-switch stop
* Delete all the logs & ovs databases::

    # rm -rf /var/log/openvswitch/*
    # rm -rf /etc/openvswitch/conf.db
* Start the OVS switches::

    # service openvswitch-switch start
* Check the ovs-vsctl, This will return empty set, except OVS ID and OVS version::

    # ovs-vsctl show
    
    [root@compute1 ~]# ovs-vsctl show 
        9f3b38cb-eefc-4bc7-828b-084b1f66fbfd
    
        ovs_version: "2.3.2"

Connect Open vSwitch with Open Daylight 
=======================================
Local IP has to be given within Open vSwitch to create tunnels. 

* Command given below is used for that purpose::

    # ovs-vsctl set Open_vSwitch <OPENVSWITCH ID> other_config:local_ip=’IP address’

* Create bridge br-ex for external traffic::

    # ovs-vsctl add-br br-ex
    # ovs-vsctl add-port br-ex eth1
* To set the manager for openvswitch::

    # ovs-vsctl set-manager tcp:10.0.0.100:6640
    This command will use ODL controller a manager for the OVS and create the br-int bridge automatically in the OVS switches.

    [root@compute1 ~]# ovs-vsctl show 
        9f3b38cb-eefc-4bc7-828b-084b1f66fbfd
        Manager "tcp:10.0.0.100:6640"
            is_connected: true
        Bridge br-int
            Controller "tcp:10.0.0.100:6653"
            fail_mode: secure
            Port br-int
                Interface br-int
        ovs_version: "2.3.2"


Configure ml2_conf.ini for Open Daylight driver
===============================================

* Edit /etc/neutron/plugins/ml2/ml2_conf.ini file in the Network node & Controller nodes only::

    type_drivers = flat,vlan,gre,vxlan
    tenant_network_types = gre,vxlan
    mechanism_drivers=opendaylight
    [ml2_type_gre]
    tunnel_id_ranges = 1:1000
    [ml2_type_vxlan]
    vni_ranges = 1:1000
    vxlan_group = 239.1.1.1
    [ml2_odl]
    password = admin
    username = admin
    url = http://10.0.0.100:8080/controller/nb/v2/neutron

Configure Neutron Database 
==========================
Neutron database has to be cleaned because of the no compatibility of Open vSwitch neutron plugin database with Open Daylight. And Open Daylight demands a clean slate of the configuration.

* SQL commands to delete & create neutron database::

    # mysql –u root –p
    # drop database neutron;
    # create database neutron;
    # grant all privileges on neutron.* to 'neutron'@'localhost' identified by 'neutron_openstack';
    # grant all privileges on neutron.* to 'neutron'@'%' identified by 'neutron_openstack';
    # exit
* To get the database schema for neutron databse::

    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

* Restart the Neutron-server:: 

    # service neutron-server start

Verify the Integration 
======================
The integration process has been completed, Now verification has to be carried out by creating the networks on Open Stack and then it is checked whether the same is reflected on Open Daylight or not. 

* Verification::

    # neutron router-create demo-router
    # neutron net-create demo-net
    # neutron subnet-create demo-net –name=demo_subnet 192.168.1.0/24
    # neutron router-interface-add demo-router demo_subnet
    # nova boot --flavor m1.tiny --image cirros-0.3.4-x86_64 --nic net-id=b680774d-69ff-4552-9676-5851f04ce812 --security-group default  demo-instance1

    +--------------------------------------+------------------------------------------------------------+
    | Property                             | Value                                                      |
    +--------------------------------------+------------------------------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                                                     |
    | OS-EXT-AZ:availability_zone          | nova                                                       |
    | OS-EXT-SRV-ATTR:host                 | -                                                          |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | -                                                          |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000037                                          |
    | OS-EXT-STS:power_state               | 0                                                          |
    | OS-EXT-STS:task_state                | scheduling                                                 |
    | OS-EXT-STS:vm_state                  | building                                                   |
    | OS-SRV-USG:launched_at               | -                                                          |
    | OS-SRV-USG:terminated_at             | -                                                          |
    | accessIPv4                           |                                                            |
    | accessIPv6                           |                                                            |
    | adminPass                            | f7D8sVB9A9Tx                                               |
    | config_drive                         |                                                            |
    | created                              | 2016-03-23T21:38:31Z                                       |
    | flavor                               | m1.tiny (1)                                                |
    | hostId                               |                                                            |
    | id                                   | 6294eebc-99d5-48f0-a22a-28315b6d61dd                       |
    | image                                | cirros-0.3.4-x86_64 (4d708949-5377-413c-ab49-6d31a5f44e7b) |
    | key_name                             | -                                                          |
    | metadata                             | {}                                                         |
    | name                                 | demo-instance1                                             |
    | os-extended-volumes:volumes_attached | []                                                         |
    | progress                             | 0                                                          |
    | security_groups                      | default                                                    |
    | status                               | BUILD                                                      |
    | tenant_id                            | ba95a008263b44759568151a773070b1                           |
    | updated                              | 2016-03-23T21:38:31Z                                       |
    | user_id                              | 0008b6dbffaf45218f94e7706e070d6b                           |
    +--------------------------------------+------------------------------------------------------------+

Network reflection on the Open Daylight
=======================================
Networks which are made on the openstack, can be seen on the Open Daylight through curl command

* Use curl command to check the networks::

    root@controller:~# curl -u admin:admin http://10.0.0.100:8181/controller/nb/v2/neutron/networks
    {
      "networks" : [ {
      "id" : "0ac8090d-ad92-4e46-b4c1-f77df9629deb",
      "tenant_id" : "d13aef590ba04caca70a00ea020b8e79",
      "name" : "demo-private",
      "admin_state_up" : true,
      "shared" : false,
      "router:external" : false,
      "provider:network_type" : "gre",
      "provider:segmentation_id" : "97",
      "status" : "ACTIVE",
      "segments" : [ ]
      }, {
      "id" : "ee477bb0-63ad-4b05-abad-f3abac812ec1",
      "tenant_id" : "d13aef590ba04caca70a00ea020b8e79",
      "name" : "Marketing",
      "admin_state_up" : true,
      "shared" : false,
      "router:external" : false,
      "provider:network_type" : "gre",
      "provider:segmentation_id" : "33",
      "status" : "ACTIVE",
      "segments" : [ ]
       } ]
    }
