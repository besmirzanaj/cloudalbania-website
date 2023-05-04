---
title: 'Free unused disk space in Virtualbox VMs'
date: "2023-05-04"
description: "Free unused disk space in Virtualbox VMs"
draft: false
tags: 
  - virtualbox
  - vmware
  - disk
  - fstrim

---

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Procedure](#procedure)
  - [On the host OS](#on-the-host-os)
  - [On the VM](#on-the-vm)

## Introduction

When using Virtualbox, after creating the VMs, and even after specifying their disks as fake SSDs to the guest OS, the TRIM operations will not run and the disk space will not get reclaimed. For example even if a guest OS reports as using 2 Gb of disk, on the host machine this will sometimes show as double the space.

To mitigate this we have to run a command from the cli to let the guest OS know that it can treat the provided disk as a SSD one.

NOTE: This guide is for Linux guest OSs, if you run this on a pre-exsisting Windows VM you will make that VM almost useless.

## Prerequisites

Any recent Virtualbox version. I am running the examples on the following version

```shell
$ vboxmanage --version
6.1.44r156814
```

The guest OS has to have the disk mounted as a SATA one. The commands below assume the OS HDD of the Guest VM is the first SATA device for that VM

## Procedure

### On the host OS

Windows

```powershell
VBoxManage storageattach "GuestOsMachineName" --storagectl "SATA" --port 0 --device 0 --nonrotational on --discard on
```

Linux

```shell
$ vboxmanage storageattach "GuestOsMachineName" --storagectl "SATA" --port 0 --device 0 --nonrotational on --discard on
```

### On the VM

Once the above is set, we can configure the guest OS to start TRIMing the SSD. The following were tested on RHEL and Ubuntu OSs

First lets enable the cron task to run TRIM command periodically

```shell
sudo systemctl enable fstrim.timer
```

NOTE: If your guest VM is installed on a LVM volume (as most distros do this by default) we need to tell LVM to issue discards:

```shell
$ sudo sed -i 's/issue_discards = 0/issue_discards = 1/g' /etc/lvm/lvm.conf
```

At this point the configuration is complete and the next time the VM's cron job for the TRIM operations will trigger it will run the TRIM command.

If we need to run immediately the TRIM command on the guest VM then we run the following

```shell
$ sudo fstrim -a
```

And that is it.