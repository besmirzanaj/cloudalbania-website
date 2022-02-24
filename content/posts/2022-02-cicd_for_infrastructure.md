---
title: 'Enabling a CI/CD infrastructure for your automation projects - Part 1'
date: "2022-02-14"
description: "automate your infrastructure configuration and management with gitlab runners"
draft: true
tags: 
  - gitlab
  - cicd
  - ansible
  - configuration
  - git
---

- [Introduction](#introduction)
- [Building our development environment](#building-our-development-environment)


## Introduction

In our day to day system administration tasks we should consider ourselves by default "lazy", meaning if we can automate simple repetitive tasks, we should do that early and gain time to learn more stuff or develop ourselves (and of course automate more stuff).

One of the best cases where automation is vital in my opinion is configuration management. In this article I will try to build an infrastructure to configure our servers automatically as soon as we build them. And the beauty of it? The server can run anywhere and we need just an SSH connection. We are going to use [Ansible](https://www.ansible.com) as our configuration management tool and GIT to store our configuration in code.

One of the steps that we are also going to do is to prepare systems to be configured with Ansible. Since ansible requires a SSH connection that can then be elevated, we will create an "ansible" user that is going 

First we will build our development environment where we are going to write the Ansible code. Then we will build a CI/CD pipeline with Gitlab and a Gitlab Runner so that we do not have to run ansible from our PC anymore and can let an automated system run it for us.

Lastly we will see how to add/remove roles for individual servers.

## Building our development environment

There is no silver line on doing this but you should at least have a BASH shell and an IDE. The IDE should have a terminal so that you could quickly run commands from there. Personally, I use Ubuntu (or Ubuntu on WSL when using Windows) and visual Studio Code as an IDE.

I will assume we have a Win 10 system so lets start by installing the Ubuntu system in the WSL. I expect the reader to have some Sysadmin experience so I am linking the official [WSL Install](https://docs.microsoft.com/en-us/windows/wsl/install) page. Choose any of the [distributions](https://docs.microsoft.com/en-us/windows/wsl/install-manual#downloading-distributions) that you are more comfortable with. By default you will get Ubuntu from the installer.

I also want my C: drive to be accessible from as `/c/` within WSL and also my Linux permissions to be [working properly](https://devblogs.microsoft.com/commandline/chmod-chown-wsl-improvements/). To achieve this I am using some custom settings in `/etc/wsl/conf` file that is configured on day 1.

```bash
bzanaj@localhost:~$ cat /etc/wsl.conf 
[automount]
root = /
enabled = true
options = "metadata,uid=1000,gid=1000,umask=002,fmask=002"
```

My `df -h` command looks like this:

```bash
bzanaj@milton:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
rootfs          124G  102G   22G  83% /
none            124G  102G   22G  83% /dev
none            124G  102G   22G  83% /run
none            124G  102G   22G  83% /run/lock
none            124G  102G   22G  83% /run/shm
none            124G  102G   22G  83% /run/user
tmpfs           124G  102G   22G  83% /sys/fs/cgroup
C:\             124G  102G   22G  83% /c
D:\             353G  247G  106G  71% /d
```

##