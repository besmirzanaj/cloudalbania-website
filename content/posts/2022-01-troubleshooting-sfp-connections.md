---
title: 'Troubleshooting SFP physical network links'
date: "2022-01-24"
description: "troubleshoot fiber and copper SFP adapter health"
draft: false
tags: 
  - sfp
  - network
  - fiber
  - nic
---

Sometimes I have faced issues in fiber connections between computers and switches and most of the time I was remote and just could not be on site. If your system is Linux based, there is a well known swiss army knife software called [ethtool](https://linux.die.net/man/8/ethtool), which I believe every sysadmin should learn to use it, that can query or control network driver and hardware settings. It can help us troubleshoot network interfaces, [and do a lot more](https://community.mellanox.com/s/article/How-to-Use-Ethtool-to-Flash-firmware).

## Basic usage of ethtool

Ethtool is my first tool when I want to diagnose the physical link of a network interface. Generally `ip link` or `ifconfig -a` will not tell you the physical parameters and the negotiation speed of the local interface and here is where I use `ethtool`.

The most basic example is to run as root (or sudo) the command with the interface name as an argument. Let's get my server's interfaces first:

```bash
root@localhost:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eno1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 14:58:d0:5f:f6:f8 brd ff:ff:ff:ff:ff:ff
3: eno2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master bond0 state UP mode DEFAULT group default qlen 1000
    link/ether 3a:25:d6:99:a6:bd brd ff:ff:ff:ff:ff:ff permaddr 14:58:d0:5f:f6:f9
4: ens2f0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 90:e2:ba:c7:6f:bc brd ff:ff:ff:ff:ff:ff
5: eno3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master bond0 state UP mode DEFAULT group default qlen 1000
    link/ether 3a:25:d6:99:a6:bd brd ff:ff:ff:ff:ff:ff permaddr 14:58:d0:5f:f6:fa
6: eno4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 14:58:d0:5f:f6:fb brd ff:ff:ff:ff:ff:ff
7: ens2f1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 90:e2:ba:c7:6f:bd brd ff:ff:ff:ff:ff:ff
    altname enp3s0f1
```

Ok, here I have 6 physical and one loopback interface. As you can see no information on link speed is available here. If I would need a bit more information on `eno1` which is `DOWN` we have this output from `ethtool`:

```bash
root@localhost:~# ethtool eno1
Settings for eno1:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Supported pause frame use: Symmetric
        Supports auto-negotiation: Yes
        Supported FEC modes: Not reported
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Advertised pause frame use: Symmetric
        Advertised auto-negotiation: Yes
        Advertised FEC modes: Not reported
        Speed: Unknown!
        Duplex: Unknown! (255)
        Auto-negotiation: on
        Port: Twisted Pair
        PHYAD: 1
        Transceiver: internal
        MDI-X: off (auto)
        Supports Wake-on: pumbg
        Wake-on: d
        Current message level: 0x00000007 (7)
                               drv probe link
        Link detected: no
```

And the output from an active connection, e.g. `eno2`, we can see this:

```bash
root@localhost:~# ethtool eno2
Settings for eno2:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Supported pause frame use: Symmetric
        Supports auto-negotiation: Yes
        Supported FEC modes: Not reported
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Advertised pause frame use: Symmetric
        Advertised auto-negotiation: Yes
        Advertised FEC modes: Not reported
        Speed: 1000Mb/s
        Duplex: Full
        Auto-negotiation: on
        Port: Twisted Pair
        PHYAD: 1
        Transceiver: internal
        MDI-X: on (auto)
        Supports Wake-on: pumbg
        Wake-on: d
        Current message level: 0x00000007 (7)
                               drv probe link
        Link detected: yes
```

Notice the amount of details that is crucial for troubleshooting, especially the `Speed: 1000Mb/s`, `Duplex`, `Port` and finally `Link detected: yes`.

## Getting SFP and media info

Let's see now how we can dig deeper in getting more information on installed SFP Ethernet modules. All SFP modules come with firmware installed that makes possible to identify themselves, pass on diagnostic info to the NIC or its driver and as often happens to set vendor hardware locks, so that a specific SFP module from a vendor to work only with this vendor's network devices (eg. switches) and not others. By the way if you need cheap SFPs that are programmed per each major vendors, just got to [FS](https://www.fs.com).

Ethtool is able to query the firmware on these modules to get that diagnostic information. The option that gets that is `--module-info`.

```bash
root@localhost:~# ethtool -h | grep module
        ethtool [ --debug MASK ][ --json ] -m|--dump-module-eeprom|--module-info DEVNAME        Query/Decode Module EEPROM information and optical diagnostics if available
```

As you can see the command now to query this information is:

```bash
root@localhost:~# ethtool --module-info <IFNAME>
```

From the above `ip link` command you can see that two of the interfaces have a different name, e.g. `ens2f1`. In my server this happens to be a 10G network interface with two NICs. The name is set by the kernel during boot time:

```bash
root@localhost:~# dmesg | grep ens2f
[    3.130365] ixgbe 0000:03:00.0 ens2f0: renamed from eth2
[    3.164304] ixgbe 0000:03:00.1 ens2f1: renamed from eth0
```

Finally the diagnostic info we need:

```bash
root@localhost:~# ethtool --module-info ens2f1
        Identifier                                : 0x03 (SFP)
        Extended identifier                       : 0x04 (GBIC/SFP defined by 2-wire interface ID)
        Connector                                 : 0x07 (LC)
        Transceiver codes                         : 0x10 0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x00
        Transceiver type                          : 10G Ethernet: 10G Base-SR
        Transceiver type                          : Ethernet: 1000BASE-SX
        Encoding                                  : 0x06 (64B/66B)
        BR, Nominal                               : 10300MBd
        Rate identifier                           : 0x02 (8/4/2G Rx Rate_Select only)
        Length (SMF,km)                           : 0km
        Length (SMF)                              : 0m
        Length (50um)                             : 80m
        Length (62.5um)                           : 30m
        Length (Copper)                           : 0m
        Length (OM3)                              : 300m
        Laser wavelength                          : 850nm
        Vendor name                               : Intel Corp
        Vendor OUI                                : 00:1b:21
        Vendor PN                                 : FTLX8571D3BCVIT1
        Vendor rev                                : A
        Option values                             : 0x00 0x3a
        Option                                    : RX_LOS implemented
        Option                                    : TX_FAULT implemented
        Option                                    : TX_DISABLE implemented
        Option                                    : RATE_SELECT implemented
        BR margin, max                            : 0%
        BR margin, min                            : 0%
        Vendor SN                                 : MVD1HJU
        Date code                                 : 160406
        Optical diagnostics support               : Yes
        Laser bias current                        : 0.394 mA
        Laser output power                        : 0.0251 mW / -16.00 dBm
        Receiver signal average optical power     : 0.0000 mW / -inf dBm
        Module temperature                        : 29.54 degrees C / 85.16 degrees F
        Module voltage                            : 3.3194 V
        Alarm/warning flags implemented           : Yes
        Laser bias current high alarm             : Off
        Laser bias current low alarm              : On
        Laser bias current high warning           : Off
        Laser bias current low warning            : On
        Laser output power high alarm             : Off
        Laser output power low alarm              : On
        Laser output power high warning           : Off
        Laser output power low warning            : On
        Module temperature high alarm             : Off
        Module temperature low alarm              : Off
        Module temperature high warning           : Off
        Module temperature low warning            : Off
        Module voltage high alarm                 : Off
        Module voltage low alarm                  : Off
        Module voltage high warning               : Off
        Module voltage low warning                : Off
        Laser rx power high alarm                 : Off
        Laser rx power low alarm                  : On
        Laser rx power high warning               : Off
        Laser rx power low warning                : On
        Laser bias current high alarm threshold   : 13.200 mA
        Laser bias current low alarm threshold    : 2.000 mA
        Laser bias current high warning threshold : 12.600 mA
        Laser bias current low warning threshold  : 3.000 mA
        Laser output power high alarm threshold   : 1.0000 mW / 0.00 dBm
        Laser output power low alarm threshold    : 0.1585 mW / -8.00 dBm
        Laser output power high warning threshold : 0.7943 mW / -1.00 dBm
        Laser output power low warning threshold  : 0.1995 mW / -7.00 dBm
        Module temperature high alarm threshold   : 78.00 degrees C / 172.40 degrees F
        Module temperature low alarm threshold    : -13.00 degrees C / 8.60 degrees F
        Module temperature high warning threshold : 73.00 degrees C / 163.40 degrees F
        Module temperature low warning threshold  : -8.00 degrees C / 17.60 degrees F
        Module voltage high alarm threshold       : 3.7000 V
        Module voltage low alarm threshold        : 2.9000 V
        Module voltage high warning threshold     : 3.6000 V
        Module voltage low warning threshold      : 3.0000 V
        Laser rx power high alarm threshold       : 1.0000 mW / 0.00 dBm
        Laser rx power low alarm threshold        : 0.0100 mW / -20.00 dBm
        Laser rx power high warning threshold     : 0.7943 mW / -1.00 dBm
        Laser rx power low warning threshold      : 0.0158 mW / -18.01 dBm
```

The most interesting fields for me while troubleshooting are the `Connector`, the two `Transceiver type` lines, `Laser wavelength`, `Vendor` and `Vendor PN`.

## Getting SFP diagnostic info from all interfaces

Instead of me trying to identify all network interfaces that have SFP connectors one by one, I can just query all of them and if there is an SFP connector, then to show me diagnostic data. Any `stderr` can be send to `/dev/null`.

The bash script I have installed on all my servers at `/usr/local/bin/` looks like this

```bash
root@localhost:~# cat /usr/local/bin/sfp_info.sh
#!/bin/sh

# sfp_info.sh
# A tool to get all diagnostic info from any SFP modules installed on a system
#
# Author: Besmir Zanaj, 2020

for IFACE in $(ls /sys/class/net/) ; do
  /sbin/ethtool --module-info ${IFACE} > /dev/null 2>&1
  if [[ $? -eq 0 ]] ; then
    echo ${IFACE}
    /sbin/ethtool --module-info ${IFACE}
  fi
  done
exit 0
```

Hope this script helps diagnosing SFP fiber issues is a bit easier now.
