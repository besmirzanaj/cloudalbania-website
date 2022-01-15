---
title: 'Automating my Homelab - Part 2 - Provisioning with Terraform'
date: "2022-01-08"
description: "VM provisioning in Proxmox with Terraform"
draft: false
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

We will start by first declaring the Proxmox provider in our `main.tf` file:

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

We then initialize this provider with the command below. Terraform will download in the background the necessary binaries in the `.terraform/providers/` folder.

```bash
$ terraform init
```

### Terraform credentials

Now we need to create a user in our Proxmox Cluster (or node) with the [proper privileges](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#creating-the-proxmox-user-and-role-for-terraform) for Terraform. This is a one time manual operation and needs to be done only once if your Proxmox hosts are in a cluster.

```bash
pveum role add TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
pveum user add terraform-prov@pve --password <password>
pveum aclmod / -user terraform-prov@pve -role TerraformProv
```

The Terraform provider for Proxmox also supports using an [API key](https://github.com/Telmate/terraform-provider-proxmox/blob/master/docs/index.md#creating-the-connection-via-username-and-api-token) rather than a password.

If we need to modify the privileges, simply issue the command showed, adding or removing privileges as needed.

```bash
pveum role modify TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
```

I chose to authenticate with a username and password on the PVE cluster so I created a `proxmox_creds.env` with this content:

```bash
export PM_USER="terraform-user@pve"
export PM_PASS="PASSWORD_HERE"
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

In my homelab I will need to manage in Proxmox both LXC containers and KVM VMs so we will need to declare both resource types.

Another topic I focused in Terraform since the beginning is to take advantage of the `for_each` loops. This allows us to declare the resources as [lists](https://www.terraform.io/language/expressions/types#list) or [maps](https://www.terraform.io/language/expressions/types#map) and Terraform will go through each items to create/destroy them.

To achieve this, in our repo, we would need several Terraform files. Namely:

1. `main.tf` - this is where the Proxmox plugin and provider will get declared. Nothing else here to keep the code tidy
2. `proxmox_homelab_vms.tf` - on this file we will configure only loops to manage VMs.
3. `proxmox_homelab_lxc.tf` - on this one we will configure any loops that are going to provision LXC containers
4. `variables.tf` - in this file we will declare all variables used in our Terraform code.
5. `terraform.tfvars` - this is the where we store actual variable values that terraform will use to grab and fill the info on the loops above or variables as needed. We will also store any other plain variable such as [strings](https://www.terraform.io/language/expressions/types#string)


The content of the files `main.tf`, `proxmox_homelab_vms.tf`, `proxmox_homelab_lxc.tf` and `terraform.tfvars` is shown above or in the examples below. The `variables.tf` file contains the variables declaration and description. In terramof you have to declare everything, including the variables. Here is part of my file:

```terraform
variable "ssh_password" {
  description = "initial ssh root password"
  type        = string
}

variable "ssh_user" {
  description = "initial ssh root user"
  type        = string
}

variable "ssh_pub_key" {
  description = "users pub lkey"
  type        = string
}

variable "k8s_masters" {
  description = "vm variables in a dictionary "
  type        = map(any)
}

variable "k8s_workers" {
  description = "vm variables in a dictionary "
  type        = map(any)
}
...
```

### VM management

As explained above, our VMs will be configured in the `proxmox_homelab_vms.tf` file and we will be assigning values to variables in `terraform.tfvars`.

As per the scope of this article, the example below will create my Kubernetes master nodes. First I declare the variables. Since I want as much separation of data from the terraform code I like to have map variable for the VMs in the `terraform.tfvars` file as below with all VM facts.

```terraform
k8s_masters = {
  m1 = { target_node = "pve02", vcpu = "4", memory = "16384", disk_size = "30G", name = "k8sm1.cloudalbania.com", ip = "192.168.88.81", gw = "192.168.88.1" },
  m2 = { target_node = "pve02", vcpu = "4", memory = "16384", disk_size = "30G", name = "k8sm2.cloudalbania.com", ip = "192.168.88.82", gw = "192.168.88.1" },
  m3 = { target_node = "pve02", vcpu = "4", memory = "16384", disk_size = "30G", name = "k8sm3.cloudalbania.com", ip = "192.168.88.83", gw = "192.168.88.1" }
}
```

Now my `proxmox_homelab_vms.tf` file to create the resources will be very generic and will create resources based on the `for_each = var.k8s_masters` loop. It will then loop up the variables of the _k8s_masters_ map and replaces them in the subsequent variable calls, for e.g `name = each.value.name`.

```terraform
resource "proxmox_vm_qemu" "rocky8-k8s-kubespray-masters" {
  for_each    = var.k8s_masters
  name        = each.value.name
  desc        = each.value.name
  target_node = each.value.target_node
  os_type     = "cloud-init"
  full_clone  = true
  memory      = each.value.memory
  sockets     = "1"
  cores       = each.value.vcpu
  cpu         = "host"
  scsihw      = "virtio-scsi-pci"
  clone       = var.k8s_source_template
  agent       = 1
  disk {
    size    = each.value.disk_size
    type    = "virtio"
    storage = "synology"
  }
  network {
    model  = "virtio"
    bridge = "vmbr1"
  }

  # Cloud-init section
  ipconfig0 = "ip=${each.value.ip}/24,gw=${each.value.gw}"
  ssh_user  = var.ssh_user
  sshkeys   = var.ssh_pub_key

  # Post creation actions
  provisioner "remote-exec" {
    inline = concat(var.extend_root_disk_script, var.firewalld_k8s_config)
    connection {
      type        = "ssh"
      user        = var.ssh_user
      password    = var.ssh_password
      private_key = file("~/.ssh/id_rsa")
      host        = each.value.ip
    }
  }
}
```

To start building the infrastructure I just need to run

```bash
$ terraform apply
```

Press `yes` when prompted and after the resources are built it will prompt with the following

```terraform
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

### Considerations when creating VMs

Some notes to consider when building VMs with the terraform provider.

#### Proxmox API is slow

Always consider the fact that the Proxmox API might be a bit slow and Terraform might get conflicting information on the status of VMs. This might lead to stuck processes or stuck VMS.To mitigate this I had to run Terraform with 3 jobs max at a time

```bash
$ terraform apply --parallelism=3 --auto-approve
```

#### Use cloud-init

We should use the setting `os_type = "cloud-init"` in our resource in order for Terraform to be able to inject later [cloud-init](https://github.com/Telmate/terraform-provider-proxmox/blob/master/docs/resources/vm_qemu.md#provision-through-cloud-init) user settings.

#### VM customizations

I am using the `remote-exec` provisioner and did not want ot have a big list of commands in this `.tf` file. You can configure the newly built VMs by calling ansible locally (will show you this in the next article) or I really wanted to run some bash scripts within the VMs. One of these important scripts is the disk resize to the new partition from the 5 GB one we crete the template with Packer. The other commands consists in firewalld configuration for these specific VMs (Opening ports for k8s)

in my `terrafrom.tfvars` file I have declared two variables with some bash shell scripts:

```terraform
firewalld_k8s_config = [
    "sudo dnf install firewalld",
    "sudo systemctl enable --now firewalld",
    "sudo firewall-cmd --permanent --add-port=6443/tcp",
    "sudo firewall-cmd --permanent --add-port=2379-2380/tcp",
    "sudo firewall-cmd --permanent --add-port=10250/tcp",
    "sudo firewall-cmd --permanent --add-port=10251/tcp",
    "sudo firewall-cmd --permanent --add-port=10252/tcp",
    "sudo firewall-cmd --permanent --add-port=10255/tcp",
    "sudo firewall-cmd --permanent --add-port=8472/udp",
    "sudo firewall-cmd --add-masquerade --permanent",
    "sudo firewall-cmd --permanent --add-port=30000-32767/tcp # only if you want NodePorts exposed on control plane IP as well",
    "sudo systemctl restart firewalld"
]
```

and

```terraform
extend_root_disk_script = [
    "sudo bash /etc/auto_resize_vda.sh"
    ]
```

Since I want all of these commands to run on my servers on server creation, Terraform offers the [`concat`](https://www.terraform.io/language/functions/concat) function to combine these two lists. This is how I am combining them in the `remote-exec`provisioner:

```terraform
provisioner "remote-exec" {
    inline = concat(var.extend_root_disk_script, var.firewalld_k8s_config)
    connection {
      type        = "ssh"
      user        = var.ssh_user
      password    = var.ssh_password
      private_key = "${file("~/.ssh/id_rsa")}"
      host        = each.value.ip
    }
  }
```

#### HDD controllers

I wanted to use a hardware controller compatible with `virtio` for HDDs and also to allow me to add more disks to the same controller in the future, rather than add more controllers per each additional disk. For this reason I added the option

```terraform
scsihw = "virtio-scsi-pci"
```

### LXC Container resource management

Same as above we want the LXC containers to be managed by a dedicated file `proxmox_homelab_lxc.tf` (this is not mandatory, we are doing this only for better readability and code organization).
Here I am showing how I am building generic Rocky Linux LXC Containers by declaring them as a `lxc_rocky_linux` map variable in `terraform.tfvars`:

```terraform
lxc_rocky_linux = {
  #lxc1 = { target_node = "pve02", name = "gitlab-runner-homelab.cloudalbania.com", memory = "2048", cores = "2", storage = "synology", disk_size = "8G", ip = "192.168.88.45", gw = "192.168.88.1", ostemplate = "synology:vztmpl/rockylinux-8-default_20210929_amd64.tar.xz" },
  #lxc2 = { target_node = "pve02", name = "ansible-runner.cloudalbania.com", memory = "2048", cores = "1", storage = "synology", disk_size = "4G", ip = "192.168.88.46", gw = "192.168.88.1", ostemplate = "synology:vztmpl/rockylinux-8-default_20210929_amd64.tar.xz" },
  #lxc3 = { target_node = "pve02", name = "besmir-lxc-test.cloudalbania.com", memory = "2048", cores = "1", storage = "synology", disk_size = "4G", ip = "192.168.88.47", gw = "192.168.88.1", ostemplate = "synology:vztmpl/rockylinux-8-default_20210929_amd64.tar.xz" },
}
```

LXC Contaier resource declaration in `proxmox_homelab_lxc.tf`:

```terraform
resource "proxmox_lxc" "rocky_linux_containers" {
  for_each    = var.lxc_rocky_linux
  target_node = each.value.target_node
  hostname    = each.value.name
  ostemplate  = each.value.ostemplate
  memory      = each.value.memory
  #cores       = each.value.cores
  unprivileged    = true
  start           = true
  password        = var.ssh_password
  ssh_public_keys = <<EOT
                    ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEAk5...
  # Terraform will crash without rootfs defined
  rootfs {
    storage = each.value.storage
    size    = each.value.disk_size
  }
  network {
    name   = "eth0"
    bridge = "vmbr1"
    ip     = "${each.value.ip}/24"
    gw     = each.value.gw
  }

  # giving some time to the container to start
  provisioner "local-exec" {
    command = "logger 'This container was built on $(date)'"
  }
}
```

again, by running `terraform apply` we will be able to see our new containers.

## Conclusion

This is the end of the second article dedicated to create and manage resources in a Proxmox environment. I am using this setup for my homelab but this can be easily extended for larger infrastructures.

The reader is urged to place the Packer and Terraform configuration in git repositories and automate the provisioning with a CI/CD platform. Personally I am using GitLab. My git repos are in https://gitlab.com and I have a [local gitlab-runner](https://gitlab.com/besmirzanaj/ansible-systems/-/blob/master/playbooks/gitlab-runner.yml) that will run the automated Terraform and Packer commands after each code change.
