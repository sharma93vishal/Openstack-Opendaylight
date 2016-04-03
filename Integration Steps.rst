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
