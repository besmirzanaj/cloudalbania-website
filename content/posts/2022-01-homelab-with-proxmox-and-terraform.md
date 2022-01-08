---
title: 'Automating my Homelab - Part 2 - Provisioning with Terraform'
date: "2022-01-07"
description: "VM provisioning in Proxmox with Terraform"
draft: true
tags: 
  - proxmox
  - packer
  - terraform
  - kubernetes
  - k8s
---

This is a continuation the previous article [Automating my Homelab - Part 1 - Building VM templates with Packer]({{< relref "/2022-01-homelab-with-proxmox-and-packer.md" >}} "Packer in Proxmox").

Now that we have the images ready to be "consumed" we need a tool to automate the task of VM creation and customization for us. ONe such tool is [Terraform](https://www.terraform.io/) which is an open-source infrastructure as code software tool that provides a CLI to manage in our case VM resources. Terraform codifies cloud APIs into declarative configuration files. In our case it will work with the [Proxmox API](https://pve.proxmox.com/wiki/Proxmox_VE_API). The best [Terraform provider](https://www.terraform.io/language/providers) as of this writing to work with the Proxmox api id developed by [Telmate](https://registry.terraform.io/namespaces/Telmate) in the [Terraform public registry](https://registry.terraform.io/providers/Telmate/proxmox/).

## Setting up the project

The repository for this project is located [here](URL HERE).

Initial steps are to create a user with the [proper privileges](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#creating-the-proxmox-user-and-role-for-terraform) for Terraform.

```console
pveum role add TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
pveum user add terraform-prov@pve --password <password>
pveum aclmod / -user terraform-prov@pve -role TerraformProv
```

The terraform provider for Proxmos also supports using an API key rather than a password

After the role is in use, if there is a need to modify the privileges, simply issue the command showed, adding or removing privileges as needed.

```console
pveum role modify TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
```

I chose to authenitcate with a username and password on the PVE cluster so I created a `proxmox_creds.env` with this content:

```console
export PM_USER="terraform-user@pve"
export PM_PASS="password"
```

Each time I will now run terrafom commands I will need to run this first:

```console
source ./proxmox_creds.env
```

[Here](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#argument-reference) is a complete argument reference for the `provider` section