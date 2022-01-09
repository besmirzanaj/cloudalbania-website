---
title: 'Automating my Homelab - Part 2 - Provisioning with Terraform'
date: "2022-01-08"
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

Now that we have the images ready to be "consumed" we need a tool to automate the task of VM creation and customization for us. One such tool is [Terraform](https://www.terraform.io/) which is an open-source infrastructure as code software tool that will allow us to manage VM resources. Terraform codifies cloud APIs into declarative configuration files. In our case it will work with the [Proxmox API](https://pve.proxmox.com/wiki/Proxmox_VE_API). The best [Terraform provider](https://www.terraform.io/language/providers) to work with the Proxmox API as of the writing of this article is developed by [Telmate](https://registry.terraform.io/namespaces/Telmate) and is available in the [Terraform public registry](https://registry.terraform.io/providers/Telmate/proxmox/).

## Setting up the project

The repository for this project is located [here](URL HERE).

We will start by first declaring the module in our `main.tf` file:

```terraform
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.9.4"
    }
  }
}

provider "proxmox" {
  pm_tls_insecure = true
  pm_api_url = "https://server1.homelab.com:8006/api2/json"
}
```

We can initialize this provider with command below, Terraform will download in the background the necessary binaries in the `.terraform/providers/` folder.

```bash
$ terraform init
```

### Terraform credentials

Now we need to create a user in our Proxmox Cluster (or node) with the [proper privileges](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#creating-the-proxmox-user-and-role-for-terraform) for Terraform.

```bash
pveum role add TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
pveum user add terraform-prov@pve --password <password>
pveum aclmod / -user terraform-prov@pve -role TerraformProv
```

The terraform provider for Proxmox also supports using an API key rather than a password

After the role is in use, if there is a need to modify the privileges, simply issue the command showed, adding or removing privileges as needed.

```bash
pveum role modify TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
```

I chose to authenticate with a username and password on the PVE cluster so I created a `proxmox_creds.env` with this content:

```bash
export PM_USER="terraform-user@pve"
export PM_PASS="password"
```

Each time I will open up the project dir I will need to source the credentials first:

```bash
source ./proxmox_creds.env
```

[Here](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#argument-reference) is a complete argument reference for the `provider` section. 

We can now start creating resources.

## Resource management

One of the best crash courses in Terraform I can suggest (which also helped me get certified as a Terraform Associate) is in this [Youtube Video](https://www.youtube.com/watch?v=SLB_c_ayRMo) from <freeCodeCamp.org>.

This helped me to move from a monolithic Terraform `main.tf` file to several split ones and also how to create complex variables such as maps and lists.

In our repo, we would need 3 Terraform files: `main.tf` where the main list of resources will get declared