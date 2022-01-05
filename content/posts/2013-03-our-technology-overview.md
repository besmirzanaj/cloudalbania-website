---
title: Our technology - An overview
date: "2013-03-18"
draft: false
tags:
    - openstack
    - cloud computing
---

We are currenty working on Openstack software and we think that at minimum we have to describe our underlying technology and make a better approach with our future customers:  
  
## Overview

OpenStack is an Infrastructure as a Service offering. It is an Open Source project, founded by RackSpace, NASA and it can be deployed as a public or private cloud.  
The OpenStack projects are: CINDER, GLANCE, KEYSTONE, NOVA, QUANTUM, SWIFT.  
  
## OpenStack Compute: (NOVA)

Project NOVA, or OpenStack Compute, provisions and manages on-demand virtual machines and associated resources (just like an VMware ESX or Hyper-V hypervisor):

* CPU
* Memory
* Disk
* Network.

Virtual machines can be started, stopped, suspended, created and deleted, while network options for a virtual machine are static, DHCP, or IPv6. The virtual machines run on hypervisors such as XEN or KVM, but others are supported too - even VMware ESXi!  
Users and administrators use the GUI to request virtual machines. To ensure a certain security level, there are security groups, similar to AWS, to control access to virtual machines and RBAC to govern user access by role and project.  
  
## Storage

### Object Storage (project SWIFT)

Object Storage is a distributed storage system for static data such as files (graphics, movies) and virtual machine images. Objects and files are written to multiple disk drives, while OpenStack is responsible for ensuring data replication and integrity. Storage scales horizontally by adding new servers. If a server or hard drive fails, OpenStack replicates its content from other active servers to new servers in the cluster. Since OpenStack uses software to ensure data replication and distribution across servers, inexpensive servers can be used rather than expensive storage hardware.  
  
### Block storage (project CINDER)

Block storage is essentially volumes used by OpenStack virtual machines. Snapshots back up data stored on block storage volumes. Snapshots can be restored or used to create a new block storage volume.  
  
### Network (project QUANTUM)

OpenStack provides networking models to accommodate different applications or users. Standard network models include flat networks or VLANs to separate servers and network traffic. OpenStack Networking manages IP addresses, to allocate static or DHCP addresses. Floating IP addresses allow traffic to be dynamically rerouted to any compute resource, for example to redirect traffic during maintenance or in the case of a failure. OpenStack Networking has an extension framework to add intrusion detection systems (IDS), load balancing, firewalls and virtual private networks (VPN) .  
  
## Shared Services

### Identity services (project KEYSTONE)

OpenStack Identity provides a central repository of users mapped to the OpenStack services they can access. OpenStack identity is a common authentication system and integrates with existing back-end directory services such as LDAP. It supports several forms of authentication including username and password, tokens and AWS (Amazon Web Services)-type logins. The identity service also provides a list of deployed services that can be queried in the OpenStack cloud and users can determine their level of access.

## OpenStack

OpenStack Administrators can:  

* Configure centralized policies across users and systems
* Create users and tenants and define permissions for compute, storage and networking resources using role-based access control (RBAC)
* Integrate with an existing directory like LDAP, allowing for a single source of identity authentication across the cloud.

## Image services (Project GLANCE)

The OpenStack Image Service provides discovery, registration and delivery services for disk and server images. Saved images can be used as a template to get new virtual servers up and running (especially useful for multiple servers of the same type and configuration). It can also be used to store and catalog an unlimited number of backups.  
The image service stores private and public images in a variety of formats:  

* AMI
* qcow2 (Qemu/KVM)
* OVF (Open Virtualization Format)
* RAW
* VDI (VirtualBox)
* VHD (Hyper-V)
* VMDK (VMWare)

![OpenStack Software Diagram](/openstack-software-diagram.png 'OpenStack Software Diagram')
