---
title: 'Limit process resources with cgroups in Linux'
date: "2022-01-24"
description: "limit process CPU usage on a linux system with cgroups"
draft: true
tags: 
  - cgroups
  - apache
  - linux
  - cpu
---

Sharing here an older article from 2017 I had written to limit the CPU usage of the `httpd` apache web server workload to not reach 100% usage and take the VM completely unreachable and useless. 

This was before I learned that I could do this with containers, which have [`cgroups`](https://en.wikipedia.org/wiki/Cgroups) as part of their construct. The article focuses on the httpd service on a CentOS 6 server but it can be extened for any kind of `init`. A [systemd](https://systemd.io/) example is included in the bottom.

Article follows

___

## Initial analysis

There are many ways where we can implement limit and throttle CPU usage.

* __nice and chrt__ are used to set scheduling priorities for processes. This is to limit their usage, not to throttle them. The processes will use as much of the CPU as the scheduler is willing to give them. An idle process (chrt -i 0) can still consume 100% CPU if there are no other processes requesting CPU time. To implement renicing of apache process a new script needs to be implemented to measure the CPU usage of every Apache process and renice it accordingly. This could affedt customer experience and as a solution was skipped.
* __cpulimit__ will throttle the CPU usage of a process, but it cannot aggregate the CPU utilization of its children processes, so the CPU will still aggregate to full usage of all cores of cpus anyway
* __cgroups__ are designed to limit and/or audit system resources. `cgroups` have the power to limit but also to throttle CPU usage ([as well as other things](https://en.wikipedia.org/wiki/Cgroups#Features)) for a process from the second it is launched. Control Groups provide a mechanism for aggregating/partitioning sets of tasks, and all their future children, into hierarchical groups with specialized behavior.

CentOS 6 provides this new kernel feature: control groups, which are called by their shorter name cgroups. Cgroups allow you to allocate resources - such as CPU time, system memory, network bandwidth, or combinations of these resources - among user-defined groups of tasks (processes) running on a system. You can monitor the cgroups you configure, deny cgroups access to certain resources, and even reconfigure your cgroups dynamically on a running system. The `cgconfig` (control group config) service can be configured to start up at boot time and re-establish your predefined cgroups, thus making them persistent across reboots.

By using cgroups, system administrators gain fine-grained control over allocating, prioritizing, denying, managing, and monitoring system resources [1]. Hardware resources can be appropriately divided up among tasks and users, increasing overall efficiency. 

## How Control Groups Are Organized
Cgroups are organized hierarchically, like processes, and child cgroups inherit some of the attributes of their parents. However, there are differences between the two models.

### The Linux Process Model
All processes on a Linux system are child processes of a common parent: the `init` process, which is executed by the kernel at boot time and starts other processes (which may in turn start child processes of their own). Because all processes descend from a single parent, the Linux process model is a single hierarchy, or tree.

Additionally, every Linux process except `init` inherits the environment (such as the PATH variable) and certain other attributes (such as open file descriptors) of its parent process.

### The Cgroup Model

Cgroups are similar to processes in that:

* they are hierarchical, and
* child cgroups inherit certain attributes from their parent cgroup.

The fundamental difference is that many different hierarchies of cgroups can exist simultaneously on a system. If the Linux process model is a single tree of processes, then the cgroup model is one or more separate, unconnected trees of tasks (i.e. processes).

Multiple separate hierarchies of cgroups are necessary because each hierarchy is attached to one or more subsystems. A subsystem represents a single resource, such as CPU time or memory. Linux kernel provides ten cgroup subsystems, listed below by name and function. a hierarchy tree for RedHat or CentOS 6 is shown below:

```console
[root@centos6-local ~]# tree /cgroup
/cgroup
├── blkio
│   ├── blkio.io_merged
│   ├── blkio.io_merged_recursive
│   ├── blkio.io_queued
│   ├── blkio.io_queued_recursive
│   ├── blkio.io_service_bytes
│   ├── blkio.io_service_bytes_recursive
│   ├── blkio.io_serviced
│   ├── blkio.io_serviced_recursive
│   ├── blkio.io_service_time
│   ├── blkio.io_service_time_recursive
│   ├── blkio.io_wait_time
│   ├── blkio.io_wait_time_recursive
│   ├── blkio.leaf_weight
│   ├── blkio.leaf_weight_device
│   ├── blkio.reset_stats
│   ├── blkio.sectors
│   ├── blkio.sectors_recursive
│   ├── blkio.throttle.io_service_bytes
│   ├── blkio.throttle.io_serviced
│   ├── blkio.throttle.read_bps_device
│   ├── blkio.throttle.read_iops_device
│   ├── blkio.throttle.write_bps_device
│   ├── blkio.throttle.write_iops_device
│   ├── blkio.time
│   ├── blkio.time_recursive
│   ├── blkio.weight
│   ├── blkio.weight_device
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── cpu
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── cpu.cfs_period_us
│   ├── cpu.cfs_quota_us
│   ├── cpu.rt_period_us
│   ├── cpu.rt_runtime_us
│   ├── cpu.shares
│   ├── cpu.stat
│   ├── limitcpu10
│   │   ├── cgroup.clone_children
│   │   ├── cgroup.procs
│   │   ├── cpu.cfs_period_us
│   │   ├── cpu.cfs_quota_us
│   │   ├── cpu.rt_period_us
│   │   ├── cpu.rt_runtime_us
│   │   ├── cpu.shares
│   │   ├── cpu.stat
│   │   ├── notify_on_release
│   │   └── tasks
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── cpuacct
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── cpuacct.stat
│   ├── cpuacct.usage
│   ├── cpuacct.usage_all
│   ├── cpuacct.usage_percpu
│   ├── cpuacct.usage_percpu_sys
│   ├── cpuacct.usage_percpu_user
│   ├── cpuacct.usage_sys
│   ├── cpuacct.usage_user
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── cpuset
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── cpuset.cpu_exclusive
│   ├── cpuset.cpus
│   ├── cpuset.effective_cpus
│   ├── cpuset.effective_mems
│   ├── cpuset.mem_exclusive
│   ├── cpuset.mem_hardwall
│   ├── cpuset.memory_migrate
│   ├── cpuset.memory_pressure
│   ├── cpuset.memory_pressure_enabled
│   ├── cpuset.memory_spread_page
│   ├── cpuset.memory_spread_slab
│   ├── cpuset.mems
│   ├── cpuset.sched_load_balance
│   ├── cpuset.sched_relax_domain_level
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── devices
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── devices.allow
│   ├── devices.deny
│   ├── devices.list
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── freezer
│   ├── cgroup.clone_children
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
├── memory
│   ├── cgroup.clone_children
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── cgroup.sane_behavior
│   ├── memory.failcnt
│   ├── memory.force_empty
│   ├── memory.kmem.failcnt
│   ├── memory.kmem.limit_in_bytes
│   ├── memory.kmem.max_usage_in_bytes
│   ├── memory.kmem.slabinfo
│   ├── memory.kmem.tcp.failcnt
│   ├── memory.kmem.tcp.limit_in_bytes
│   ├── memory.kmem.tcp.max_usage_in_bytes
│   ├── memory.kmem.tcp.usage_in_bytes
│   ├── memory.kmem.usage_in_bytes
│   ├── memory.limit_in_bytes
│   ├── memory.max_usage_in_bytes
│   ├── memory.memsw.failcnt
│   ├── memory.memsw.limit_in_bytes
│   ├── memory.memsw.max_usage_in_bytes
│   ├── memory.memsw.usage_in_bytes
│   ├── memory.move_charge_at_immigrate
│   ├── memory.oom_control
│   ├── memory.pressure_level
│   ├── memory.soft_limit_in_bytes
│   ├── memory.stat
│   ├── memory.swappiness
│   ├── memory.usage_in_bytes
│   ├── memory.use_hierarchy
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
└── net_cls
    ├── cgroup.clone_children
    ├── cgroup.procs
    ├── cgroup.sane_behavior
    ├── net_cls.classid
    ├── notify_on_release
    ├── release_agent
    └── tasks
```
 
### Available Subsystems in the Linux kernel

* `blkio` - this subsystem sets limits on input/output access to and from block devices such as physical drives (disk, solid state, or USB).
* `cpu` - this subsystem uses the scheduler to provide cgroup tasks access to the CPU.
* `cpuacct` — this subsystem generates automatic reports on CPU resources used by tasks in a cgroup.
* `cpuset` — this subsystem assigns individual CPUs (on a multicore system) and memory nodes to tasks in a cgroup.
* `devices` — this subsystem allows or denies access to devices by tasks in a cgroup.
* `freezer` — this subsystem suspends or resumes tasks in a cgroup.
* `memory` — this subsystem sets limits on memory use by tasks in a cgroup and generates automatic reports on memory resources used by those tasks.
* `net_cls` — this subsystem tags network packets with a class identifier (classid) that allows the Linux traffic controller (tc) to identify packets originating from a particular cgroup task.
* `net_prio` — this subsystem provides a way to dynamically set the priority of network traffic per network interface.
* `ns` — the namespace subsystem.

### CFS Tunable Parameters
_Completely Fair Scheduler_ ([CFS](https://en.wikipedia.org/wiki/Completely_Fair_Scheduler)) — a proportional share scheduler which divides the CPU time (CPU bandwidth) proportionately between groups of tasks (cgroups) depending on the priority/weight of the task or shares assigned to cgroups.

In _CFS_, a cgroup can get more than its share of CPU if there are enough idle CPU cycles available in the system, due to the work conserving nature of the scheduler. This is usually the case for cgroups that consume CPU time based on relative shares. Ceiling enforcement can be used for cases when a hard limit on the amount of CPU that a cgroup can utilize is required (that is, tasks cannot use more than a set amount of CPU time).

The following options can be used to configure ceiling enforcement or relative sharing of CPU:

#### Ceiling Enforcement Tunable Parameters

`cpu.cfs_period_us` specifies a period of time in microseconds ("µs" are represented here as "us") for how regularly a cgroup's access to CPU resources should be reallocated. If tasks in a cgroup should be able to access a single CPU for 0.2 seconds out of every 1 second, set `cpu.cfs_quota_us` to 200000 and `cpu.cfs_period_us` to 1000000. 

The upper limit of the `cpu.cfs_quota_us` parameter is 1 second and the lower limit is 1000 microseconds. `cpu.cfs_quota_us` specifies the total amount of time in microseconds for which all tasks in a cgroup can run during one period (as defined by `cpu.cfs_period_us`).
As soon as tasks in a cgroup use up all the time specified by the quota, they are throttled for the remainder of the time specified by the period and not allowed to run until the next period. 

If tasks in a cgroup should be able to access a single CPU for 0.2 seconds out of every 1 second, set `cpu.cfs_quota_us` to 200000 and `cpu.cfs_period_us` to 1000000. Note that the quota and period parameters operate on a CPU basis. 

To allow a process to fully utilize two CPUs, for example, set `cpu.cfs_quota_us` to 200000 and `cpu.cfs_period_us` to 100000. Setting the value in `cpu.cfs_quota_us` to `-1` means that the cgroup does not adhere to any CPU time restrictions. This is also the default value for every cgroup (except the root cgroup). 

`cpu.stat` reports CPU time statistics using the following values:

* `nr_periods` — number of period intervals (as specified in cpu.cfs_period_us) that have elapsed.
* `nr_throttled` — number of times tasks in a cgroup have been throttled (that is, not allowed to run because they have exhausted all of the available time as specified by their quota).
* `throttled_time` — the total time duration (in nanoseconds) for which tasks in a cgroup have been throttled.

## Limitig apache process to 80% CPU usage
Limiting apache and its child process can be implemented by putting all child processes into a custom control group and aggregate the total resources availabe.

Let's test with a VM with 4 vCPUs. Initially we need a control group with groups options to limit the total CPU utilization to 80%. This is just a placeholder control group, we're still not using it. We need 80% of (4 * 100%) CPUs = 320. This is the number of maximum CPU usage we should expect from 80% limitation in a 4 CPU system 320/400 = 0.8

The `/etc/cgconfig.d/` directory is reserved for storing configuration files for specific applications and use cases. These files should be created with the _.conf_ suffix and adhere to the same syntax rules as `/etc/cgconfig.conf`. Translating this into groups, the config will be as follows and located in `/etc/cgconfig.d/limitcpu80.conf`

```console
[root@centos6-server ~]# cat /etc/cgconfig.d/limitcpu80.conf
#limit to 80% cpu
group limitcpu80{
    cpu {
        # Limit a destination to 80% of 4 CPU.
        cpu.cfs_quota_us = "32000";
        cpu.cfs_period_us = "10000";
    }
}
```

To make sure we are limiting apache services we will need to start this service in a cgroup. Services that can be started in cgroups must

* use a `/etc/sysconfig/servicename` file, or
* use the _daemon()_ function from `/etc/init.d/functions` to start the service

To make an eligible service start in a cgroup, we will edit its file in the `/etc/sysconfig` directory to include an entry in the form `CGROUP_DAEMON="cpu:limitcpu80"` where subsystem is a subsystem associated with a particular hierarchy in this case "cpu", and `control_group` is our cgroup in that hierarchy.  

## Implementation in Puppet configuration management

In your local `puppet/http` module you can include the following two files. 


__template: limitcpu80.conf.erb__
```puppet
#limit to 80% cpu
group limitcpu80 {
    cpu {
        # Limit a destination to 80% CPU usage.
        cpu.cfs_quota_us = "<%= @cfs_quota_us %>";
        cpu.cfs_period_us = "<%= @cfs_period_us %>";
    }
}
```

Since the number of CPUs in a VM can vary we need to implement a formula for the `cfs_quota_us` variable. Initally we define the `cpu_factor` which will get us the factor of 80% of all CPUs in a given system. 

```console
$cpu_factor    = $facts['processors']['count']*0.8*10000
````

The actual class now will look like this

__class manifest: cgrouplimitcpu80.pp__
```puppet
class httpd::cgrouplimitcpu80 (
  Boolean $cgroups_limit_enabled = $httpd::cgroups_limit_enabled,
  $cpu_factor    = $facts['processors']['count']*0.8*10000
  $cfs_quota_us  = inline_template('<%= @cpu_factor.to_i %>')
  $cfs_period_us = 10000,
  ){
  if $cgroups_limit_enabled {
    package { 'libcgroup':
      ensure => installed,
      }
    service { 'cgconfig':
      ensure    => running,
      subscribe => Package['libcgroup'],
      enable    => true,
    }
    service { 'cgred':
      ensure    => running,
      subscribe => [ Package['libcgroup'], Service['cgconfig'], ],
      enable    => true,
    }
    file { '/etc/cgconfig.d/limitcpu80.conf':
      ensure  => file,
      content => template('httpd/limitcpu80.conf.erb'),
    }
    Service['cgred'] ~> Service['httpd']
  }
  # Disabled by default
  else {
    service { 'cgconfig':
      ensure => stopped,
      enable => false,
    }
    service { 'cgred':
      ensure => stopped,
      enable => false,
    }
  Service['cgred'] ~> Service['httpd']
  }
}
```

Enabling now this cgroup is done at the `/etc/syscinfig/httpd` file. The definition of `/etc/syscinfig/httpd` can be stored in your templates folder.

__template/sysconfig/httpd.erb__
```puppet
...
<% if @cgroups_limit_enabled -%>
# Starting a Service in a Control Group
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/starting_a_process
<% unless @cgroup_daemon.nil? -%>
CGROUP_DAEMON="<%= @cgroup_daemon %>"
<% else -%>
#CGROUP_DAEMON=
<% end -%>
<% end -%>
```

Now we can update files in hiera by including the new class in the desired profile with

```puppet
class { '::httpd::cgrouplimitcpu80': }
```

Then to enable cgroups for apache you should specify the feature in the respective hierarchy file eg:

```yaml
httpd::cgroups_limit_enabled: true
```

## Benchmarking

Two sets of benchmarking were performed with [`ab`](https://httpd.apache.org/docs/2.4/programs/ab.html) to test the cgroups implementation:

1. Full throttle
2. Normal average traffic

Monitoring of the httpd process was performed by the following command

```console
[root@centos6-server ~]# while true ; do top -bn1 | awk '$12=="httpd" {s+=$9;} END {print s "% CPU";}' | column -t; done
```

### Full throttle testing
Apache benchmark was issued with the following parameters to allow keepalives with 300 concurrent connections for 10,000,000 queries. Test was performed from between two servers, one client and one running `httpd` with a 10 Gbps connection and with a longer period of time simulating a real case scenario.

```console
[root@centos6-local ~]# ab -n 10000000 -c 300 -k -H "Host: my.host.name" http://my.ip.add.ress/
```

### Test results
Apache benchmark results with and without cgroups. There is already lower CPU usage:

__without cgroups__

* load max: 247.51, 216.04, 160.03
* max httpd CPU: __381% __CPU

__with cgroups__

* load max: 249.88, 221.50, 137.20
* max httpd CPU: __303%__ CPU

It was also noticed that during benchmarks with `ab`, a high cpu time was used by the `cgrules` daemon itself thereof contributing to a slight higher total overall load:

```console
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND 
3893 root     20   0 19160 6076 1484 R 97.0  0.0   1:07.47 cgrulesengd                     
``` 

### Normal Average testing

From a production Load Balancer in production I was able to get the average concurrent connections for a web server and it was ~75. Then I run the benchmark again for ~2 hrs.

```console
[root@centos6-server ~]# ab -n 10000000 -c 75 -H "Host: my.host.name" http://my.ip.add.ress/
```

the load is pretty stable and in the same level of servers:

```console
[root@centos6-server ~]# uptime
13:17:57 up 1 day, 1:26, 1 user, load average: 2.77, 2.62, 2.33
```

The cgroups daemon "cgrulesengd" now reports ~ 11% cpu usage

```console
PID USER      PR  NI  VIRT  RES  SHR  S %CPU %MEM    TIME+    COMMAND 
7 root        20  0   19000 6000 1520 S 11.3 0.0     14:35.86 cgrulesengd
```

To find out the number of cgroups is throttling the cpu we can check the cpu.stat file in

```console
[root@centos6-server ~]# cat /cgroup/cpu/limitcpu80/cpu.stat
nr_periods 84017
nr_throttled 12198
throttled_time 65691839723
```

## Cgroups Examples

### Example CPU limits usage

The following examples assume you have an existing hierarchy of cgroups configured and the CPU subsystem mounted on your system.

To allow one cgroup to use 25% of a single CPU and a different cgroup to use 75% of that same CPU, use the following commands:

```console
[root@centos6-server ~]# echo 250 > /cgroup/cpu/blue/cpu.shares
[root@centos6-server ~]# echo 750 > /cgroup/cpu/red/cpu.shares
```

To limit a cgroup to fully utilize a single CPU, use the following commands:

```console
[root@centos6-server ~]# echo 10000 > /cgroup/cpu/red/cpu.cfs_quota_us
[root@centos6-server ~]# echo 10000 > /cgroup/cpu/red/cpu.cfs_period_us
```

To limit a cgroup to utilize 10% of a single CPU, use the following commands:

```console
[root@centos6-server ~]# echo 10000 > /cgroup/cpu/red/cpu.cfs_quota_us
[root@centos6-server ~]# echo 100000 > /cgroup/cpu/red/cpu.cfs_period_us
```

On a multi-core system, to allow a cgroup to fully utilize two CPU cores, use the following commands:

```console
[root@centos6-server ~]# echo 200000 > /cgroup/cpu/red/cpu.cfs_quota_us
[root@centos6-server ~]# echo 100000 > /cgroup/cpu/red/cpu.cfs_period_us
```

### Cgroups considerations
In regards to cgroups implementation we should take into consideration the fact the now apache will use maximum of 80% of CPU so the relative monitoring in place should be adapted to catch when apache will reach this new limit limit.

## Systemd example

If we had to implement the same limitation with systemd init then instead we would need to create a drop-in systemd file with the cgroups configs. The implementation is with cgroups2 so the syntax is a bit different. We do not need to calculate anymore the ration of `cpu.cfs_quota_us`/`cpu.cfs_period_us`. An implementation example on a CentOS 7 system could look something like this:

```console
[root@centos7-server ~]# cat/etc/systemd/system/multi-user.target.wants/httpd.service.d/limitcpu80.conf
[Service]
CPUAccounting=True
CPUQuota=80%
```

## References
1. [INTRODUCTION TO CONTROL GROUPS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01)
2. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01#sec-How_Control_Groups_Are_Organized
3. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-relationships_between_subsystems_hierarchies_control_groups_and_tasks
4. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpuacct
5. http://manpages.ubuntu.com/manpages/yakkety/man5/cgconfig.conf.5.html
6. http://kennystechtalk.blogspot.ca/2015/04/throttling-cpu-usage-with-linux-cgroups.html
