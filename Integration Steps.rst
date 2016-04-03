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

+ 1. Open Daylight software runs on platform independent Java virtual machine and can be installed on any Operating system on which Java is installed.
+ 2. The basic requirement is to install it on a system with a multi-core processor and 8 GB RAM in order to get optimum results.

Command to install java jdk on Linux (Ubuntu)
**#apt-get install openjdk-7-jdk**

Download the latest version of Open Daylight from their `official website <https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.4.0-Beryllium/distribution-karaf-0.4.0-Beryllium.tar.gz>`_ by using wget command.
 
Uncompress the packages : **#tar -xvf distribution-karaf-0.4.0-Beryllium.tar.gz**

To use the OpenFlow version 1.3, make the changes in the file **etc/custom.properties** in line number 82. Make sure this line is uncommented.

**ovsdb.of.version=1.3**

Start the controller: **./bin/start**
Controller will start subsequently, in case ports have to be checked, **netstat –ntl** is the command.
To take the access of controller as a client: 
**./bin/client –u karaf**

.. Image:: https://github.com/sharma93vishal/Openstack-Opendaylight/blob/master/Images/Picture1.png

To use Open Daylight with Open Stack, the features of Open Stack have to be installed on the controller.

**opendaylight-user@root>feature:install odl-ovsdb-openstack odl-dlux-all**

Command given above will install all the features to connect with Open Stack.
After this, to check whether everything works fine, use curl command. It will show empty network list.

**curl -u admin:admin http://10.0.0.100:8181/controller/nb/v2/neutron/networks**

Erase all VMs, Networks, Routers & Ports in Controller Node 
===========================================================

In order to allow Open Daylight to work with Open Stack, all the Virtual Machines, Routers, Networks, Subnets and ports are needed to be removed.
Following steps will guide you through the cleaning process.

* Delete all the instances::
 **# nova delete <instance names>**
* Remove all the routers::
 **# neutron router-delete <router name>**
* Delete all the subnets & networks::
 **# neutron subnet-delete <subnet name>**
 **# neutron net-delete <net name>**
* Check that all ports have been cleaned – at this point, this should be an empty list::
 **# neutron port-list**
* Stop the neutron-server::
 To avoid the conflict between Neutron and Open Daylight, neutron-server has to be shutdown.
 
 **# service neutron-server stop**

Open vSwitches in Compute & Network node 
========================================
The Neutron OVS plugin has to be deleted from compute & Network node because Neutron is not handling OVS switches no more. So all the configurations of the OVS switches are needed to be cleaned.

* Delete the neutron ovs-plugin agent::
 **# apt-get purge neutron-plugin-openvswitch-agent**
* Stop the OVS switches::
 **# service openvswitch-switch stop**
* Delete all the logs & ovs databases::
 **# rm -rf /var/log/openvswitch/***
 
 **# rm -rf /etc/openvswitch/conf.db**
* Start the OVS switches::
 **# service openvswitch-switch start**
* Check the ovs-vsctl, This will return empty set, except OVS ID and OVS version::
 **# ovs-vsctl show**

Connect Open vSwitch with Open Daylight 
=======================================
Local IP has to be given within Open vSwitch to create tunnels. Command given below is used for that purpose.

**# ovs-vsctl set Open_vSwitch <OPENVSWITCH ID> other_config:local_ip=’IP address’**

* Create bridge br-ex for external traffic::

 # ovs-vsctl add-br br-ex
 # ovs-vsctl add-port br-ex eth1
* To set the manager for openvswitch::
 # ovs-vsctl set-manager tcp:10.0.0.100:6640
 
 This command will use ODL controller a manager for the OVS and create the br-int bridge automatically in the OVS switches, high level control flow is given below, to explain the methodology.

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
