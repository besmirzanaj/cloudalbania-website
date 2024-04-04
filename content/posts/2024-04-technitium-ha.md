---
title: 'Setup an HA Technitium DNS cluster at home'
date: "2024-04-03"
description: "Setup an HA Technitium DNS cluster at home"
draft: true
tags: 
  - technitium
  - homelab
  - dns
  - dhcp

---

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Procedure](#procedure)
  - [On the host OS](#on-the-host-os)
  - [On the VM](#on-the-vm)

## Introduction

[Technitium DNS Server](https://technitium.com/dns/) is a really interesting DNS server that you can use in your homelab. I have gone from [pi-hole ](https://pi-hole.net/) to [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) and while I really liked the minimalistic design and footprint of AdGuard Home, it had some issues when creating custom DNS records to use in a more advanced homelab scenario. The main painpoint for me was terraform integration. While there are both terraform plugins for [pihole](https://registry.terraform.io/providers/ryanwholey/pihole/latest/docs/resources/dns_record) and [AdGuard Home](https://registry.terraform.io/providers/gmichels/adguard/latest/docs/resources/user_rules) using them gave me stability issues in pihole and never working in AdGuard Home.

After looking for a new solution I found in a Reddit post about Technitium and that it is an open source authoritative as well as recursive DNS server.

In this article I am going to install technitium in a docker container as a primary DNS server and on a RaspberryPi as a backup DNS server. After settting these up, I will enable a `cron` job on the secondary DNS server to pull and sync the configurations from the primary DNS server periodically to keep them in sync.

## Prerequisites for Technitium DNS

Any recent OS should work. I am focusing on Linux systems in this article. Make sure to install `gzip` and `tar` beforehand as the installer script requires these packages.

You can install it as a systemd daemon (on a VM or a raspberrypi) or use the provided [`docker-compose.yml`](https://github.com/TechnitiumSoftware/DnsServer/blob/master/docker-compose.yml) file. To use `docker` and `docker-compose` (or `podman`) you need to [install](https://docs.docker.com/engine/install/) them first ([docker-compose install document](https://docs.docker.com/compose/install/linux/)).

## Technitium installation procedure

For the sake of the examples, let's assume the primary server has an IP address `192.168.100.5` and hostname `dns01.home.arpa` and the secondary server (our raspberrypi) has an IP address `192.168.100.6` and hostname `dns02.home.arpa`.

NOTE: this article will not describe how to configure technitium.

### On the docker host

Let's create a `docker-compose.yml` file from the suggested git repo. I am using Technitium also as a DHCP server, so I need to configure docker network in `host` mode. I am using [`home.arpa`](https://datatracker.ietf.org/doc/html/rfc8375.html) domain for the examples.

```yml filename=docker-compose.yml
name: technitium
services:
  dns-server:
    cpu_shares: 50
    container_name: dns-server
    deploy:
      resources:
        limits:
          memory: 512M
    hostname: dns-server
    image: technitium/dns-server:latest
    network_mode: "host"
    restart: unless-stopped
    volumes:
      - type: bind
        source: /local/data/path
        target: /etc/dns
    environment:
      - DNS_SERVER_DOMAIN=dns01.home.arpa
      - DNS_SERVER_ADMIN_PASSWORD=password
```

and thats it. Lets start our service now:

```console
[root@local-box technitium]# podman-compose up
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.6.1
** excluding:  set()
['podman', 'ps', '--filter', 'label=io.podman.compose.project=technitium', '-a', '--format', '{{ index .Labels "io.podman.compose.config-hash"}}']
podman create --name=dns-server --label io.podman.compose.config-hash=6353e5665c48089d11116fbdc4f18eec215d2852bd47ac7e2498c1c7d2aadaa4 --label io.podman.compose.project=technitium --label io.podman.compose.version=1.0.6 --label PODMAN_SYSTEMD_UNIT=podman-compose@technitium.service --label com.docker.compose.project=technitium --label com.docker.compose.project.working_dir=/root/tmp/technitium --label com.docker.compose.project.config_files=docker-compose.yml --label com.docker.compose.container-number=1 --label com.docker.compose.service=dns-server -e DNS_SERVER_DOMAIN=dns01.home.arpa -e DNS_SERVER_ADMIN_PASSWORD=password -v /local/data/path:/etc/dns --network host --hostname dns-server --restart unless-stopped --cpu-shares 50 -m 512m technitium/dns-server:latest
✔ docker.io/technitium/dns-server:latest
Trying to pull docker.io/technitium/dns-server:latest...
Getting image source signatures
Copying blob 0b3a32d011fb done
Copying blob 4d3bcc6169b1 done
Copying blob 1b5cdfe08966 done
Copying blob 8a1e25ce7c4f skipped: already exists
Copying blob d038975ffc90 done
Copying blob e324257af4ea done
Copying blob 30cc68ebc435 done
Copying blob 6bbeccd44fb6 done
Copying blob 34c19fcb0c16 done
Copying blob 904799a03ea6 done
Copying config d7ec3f033a done
Writing manifest to image destination
76c54e7b879cc31be0581124efc5a6ab8f4a2bfc4ab07f8cfcc55e64dc40ca83
exit code: 0
podman start -a dns-server
[dns-server] | Technitium DNS Server was started successfully.
[dns-server] | Using config folder: /etc/dns
[dns-server] |
[dns-server] | Note: Open http://dns-server:5380/ in web browser to access web console.
[dns-server] |
[dns-server] | Press [CTRL + C] to stop...
```

An there you go. Now we can open the Technitium web UI and continue to configure it from there. To run this as a daemon, just add `-d` to your compose command.


### On the raspberry pi host

First make sure to not use the builtin system-resolved package as it will give you issues staring the server:

```console
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

Then follow the [official guide](https://blog.technitium.com/2017/11/running-dns-server-on-ubuntu-linux.html) to install technitium. Basically:

```console
root@dns02.home.arpa:~# curl -sSL https://download.technitium.com/dns/install.sh | sudo bash

===============================
Technitium DNS Server Installer
===============================

Installing ASP.NET Core Runtime...
ASP.NET Core Runtime was installed successfully!

Downloading Technitium DNS Server...
Updating Technitium DNS Server...
Configuring systemd service...

Technitium DNS Server was installed successfully!
Open http://dns02.home.arpa:5380/ to access the web console.

Donate! Make a contribution by becoming a Patron: https://www.patreon.com/technitium

root@dns02.home.arpa:~# 
```

If you are using 
### Enable dual DNS servers in the DHCP scope

In the DHCP 

## Technitium HA
