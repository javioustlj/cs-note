---
title: operation
weight: 100
date: 2025-09-15
description:
    应用与实践
---


```sh
$ ibstat
```
可以看到每一个网卡的
```
CA 'mlx5_0'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a256c
        System image GUID: 0x5c257303007a256c
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a256c
                Link layer: Ethernet
CA 'mlx5_1'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a23ec
        System image GUID: 0x5c257303007a23ec
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a23ec
                Link layer: Ethernet
CA 'mlx5_2'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a24a0
        System image GUID: 0x5c257303007a24a0
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a24a0
                Link layer: Ethernet
CA 'mlx5_3'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a2584
        System image GUID: 0x5c257303007a2584
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a2584
                Link layer: Ethernet
CA 'mlx5_4'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a2468
        System image GUID: 0x5c257303007a2468
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a2468
                Link layer: Ethernet
CA 'mlx5_5'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x9c63c0030008c93a
        System image GUID: 0x9c63c0030008c93a
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 100
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x9e63c0fffe08c93a
                Link layer: Ethernet
CA 'mlx5_6'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a25cc
        System image GUID: 0x5c257303007a25cc
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a25cc
                Link layer: Ethernet
CA 'mlx5_7'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a25a8
        System image GUID: 0x5c257303007a25a8
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a25a8
                Link layer: Ethernet
CA 'mlx5_8'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.39.3004
        Hardware version: 0
        Node GUID: 0x5c257303007a27f0
        System image GUID: 0x5c257303007a27f0
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x5e2573fffe7a27f0
                Link layer: Ethernet

```


使用这个命令可以查看 InfiniBand 设备与 Linux 网络接口的对应关系：
```sh
$ ibdev2netdev
```

```txt
mlx5_0 port 1 ==> ens13np0 (Up)
mlx5_1 port 1 ==> ens14np0 (Up)
mlx5_2 port 1 ==> ens11np0 (Up)
mlx5_3 port 1 ==> ens12np0 (Up)
mlx5_4 port 1 ==> ens15np0 (Up)
mlx5_5 port 1 ==> ens22np0 (Up)
mlx5_6 port 1 ==> ens16np0 (Up)
mlx5_7 port 1 ==> ens17np0 (Up)
mlx5_8 port 1 ==> ens18np0 (Up)
```

查看Linux网络接口状态
```sh
$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens13np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:25:6c brd ff:ff:ff:ff:ff:ff
3: ens14np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:23:ec brd ff:ff:ff:ff:ff:ff
4: ens11np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:24:a0 brd ff:ff:ff:ff:ff:ff
5: ens12np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:25:84 brd ff:ff:ff:ff:ff:ff
6: ens15np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:24:68 brd ff:ff:ff:ff:ff:ff
7: ens22np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 9c:63:c0:08:c9:3a brd ff:ff:ff:ff:ff:ff
8: ens16np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:25:cc brd ff:ff:ff:ff:ff:ff
9: ens17np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:25:a8 brd ff:ff:ff:ff:ff:ff
10: ens18np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 4200 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 5c:25:73:7a:27:f0 brd ff:ff:ff:ff:ff:ff
11: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:ff:2c:fe:9c brd ff:ff:ff:ff:ff:ff
```

```sh
$ sudo ethtool ens13np0
```
```txt
Settings for ens13np0:
        Supported ports: [ FIBRE ]
        Supported link modes:   1000baseT/Full
                                1000baseKX/Full
                                10000baseT/Full
                                10000baseKR/Full
                                10000baseR_FEC
                                40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
                                25000baseCR/Full
                                25000baseKR/Full
                                25000baseSR/Full
                                50000baseCR2/Full
                                50000baseKR2/Full
                                100000baseKR4/Full
                                100000baseSR4/Full
                                100000baseCR4/Full
                                100000baseLR4_ER4/Full
                                50000baseSR2/Full
                                1000baseX/Full
                                10000baseCR/Full
                                10000baseSR/Full
                                10000baseLR/Full
                                10000baseER/Full
                                50000baseKR/Full
                                50000baseSR/Full
                                50000baseCR/Full
                                50000baseLR_ER_FR/Full
                                50000baseDR/Full
                                100000baseKR2/Full
                                100000baseSR2/Full
                                100000baseCR2/Full
                                100000baseLR2_ER2_FR2/Full
                                100000baseDR2/Full
                                200000baseKR4/Full
                                200000baseSR4/Full
                                200000baseLR4_ER4_FR4/Full
                                200000baseDR4/Full
                                200000baseCR4/Full
        Supported pause frame use: Symmetric
        Supports auto-negotiation: Yes
        Supported FEC modes: None BaseR RS
        Advertised link modes:  1000baseT/Full
                                1000baseKX/Full
                                10000baseT/Full
                                10000baseKR/Full
                                10000baseR_FEC
                                40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
                                25000baseCR/Full
                                25000baseKR/Full
                                25000baseSR/Full
                                50000baseCR2/Full
                                50000baseKR2/Full
                                100000baseKR4/Full
                                100000baseSR4/Full
                                100000baseCR4/Full
                                100000baseLR4_ER4/Full
                                50000baseSR2/Full
                                1000baseX/Full
                                10000baseCR/Full
                                10000baseSR/Full
                                10000baseLR/Full
                                10000baseER/Full
                                50000baseKR/Full
                                50000baseSR/Full
                                50000baseCR/Full
                                50000baseLR_ER_FR/Full
                                50000baseDR/Full
                                100000baseKR2/Full
                                100000baseSR2/Full
                                100000baseCR2/Full
                                100000baseLR2_ER2_FR2/Full
                                100000baseDR2/Full
                                200000baseKR4/Full
                                200000baseSR4/Full
                                200000baseLR4_ER4_FR4/Full
                                200000baseDR4/Full
                                200000baseCR4/Full
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes
        Advertised FEC modes: RS
        Speed: 200000Mb/s
        Duplex: Full
        Port: FIBRE
        PHYAD: 0
        Transceiver: internal
        Auto-negotiation: on
        Supports Wake-on: d
        Wake-on: d
        Current message level: 0x00000004 (4)
                               link
        Link detected: yes
```

```sh
$ sudo mlxfwmanager
Querying Mellanox devices firmware ...

Device #1:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:3b:00.0
  Base MAC:         5c25737a23ec
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #2:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:cb:00.0
  Base MAC:         5c25737a25a8
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #3:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:5d:00.0
  Base MAC:         5c25737a2584
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #4:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:4b:00.0
  Base MAC:         5c25737a24a0
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #5:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:db:00.0
  Base MAC:         5c25737a27f0
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #6:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:18:00.0
  Base MAC:         5c25737a256c
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #7:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:9b:00.0
  Base MAC:         5c25737a2468
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #8:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-ECA_Ax
  Description:      ConnectX-6 VPI adapter card; 100Gb/s (HDR100; EDR IB and 100GbE); single-port QSFP56; PCIe3.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000222
  PCI Device Name:  0000:a8:00.0
  Base MAC:         9c63c008c93a
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found

Device #9:
----------

  Device Type:      ConnectX6
  Part Number:      MCX653105A-HDA_Ax
  Description:      ConnectX-6 VPI adapter card; HDR IB (200Gb/s) and 200GbE; single-port QSFP56; PCIe4.0 x16; tall bracket; ROHS R6
  PSID:             MT_0000000223
  PCI Device Name:  0000:bb:00.0
  Base MAC:         5c25737a25cc
  Versions:         Current        Available
     FW             20.39.3004     N/A
     PXE            3.7.0300       N/A
     UEFI           14.32.0017     N/A

  Status:           No matching image found
```


```sh
$ sudo mst status
MST modules:
------------
    MST PCI module is not loaded
    MST PCI configuration module is not loaded

PCI Devices:
------------

3b:00.0

cb:00.0

5d:00.0

4b:00.0

db:00.0

18:00.0

9b:00.0

a8:00.0

bb:00.0
```
