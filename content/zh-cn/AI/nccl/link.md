---
title: link
linktitle: link
weight: 2
date: 2025-10-30
description:
  NCCL中的link类型
---

## 总体介绍

NCCL中共有10种link类型：
```cpp
// We want link types and path types to match as much as possible
#define LINK_LOC 0
#define LINK_NVL 1
// Skipping 2 for PATH_NVB
#define LINK_C2C 3
#define LINK_PCI 4
// Skipping 5 for PATH_PXB
// Skipping 6 for PATH_PXN
// Skipping 7 for PATH_P2C
// Skipping 8 for PATH_PHB
#define LINK_SYS 9
#define LINK_NET 10
```

注：NCCL 中链路的带宽（link->bw）通常以 GB/s 表示，拓扑代码用这些链路类型和带宽来构建路径、估算传输成本并选择通信策略（例如是否走 NVLink、是否跨 NUMA、是否发网卡等）。


## LINK_LOC (0) — 本地（Local）


**含义**：表示“本地自身”或不需要跨设备/总线的连接。

**典型场景**：节点自身（node 指向自己），或者用于占位表示“无传输成本/本地访问”。

**带宽**：通常被视为无限大或非常高（本地访问成本可忽略）。

**用途**：表示路径的起点/终点在同一节点/同一设备时使用；在路径计算里常作为最低成本的特殊情况（PATH_LOC）。

## LINK_NVL (1) — NVLink（或设备间高速互联）

**含义**：GPU 之间直接的高速互联（NVLink / NVSwitch / GPU 借助专用互连的链路）。

**典型场景**：同主机内两块支持 NVLink 的 GPU 之间的连接；也可表示通过 NVSwitch 的连接。

**带宽**：使用与 GPU 架构相关的 NVLink 带宽（代码中通过 ncclTopoNVLinkBw(cudaCompCap) 返回，比如 SM70/SM80 等不同值）。

**用途**：优先选择此类链路进行 GPU-GPU 通信以获得最低延迟和最高带宽（PATH_NVL）。对 NCCL 算法选择非常关键（直接 P2P 或 NVLS 模式等）。

## LINK_C2C (3) — C2C（CPU↔GPU / 通过主机内存的拷贝路径）

**含义**：表示“C2C”类型的链路，语义上是 GPU 与 CPU（NUMA）或通过主机内存/CPU 的拷贝通道（也可理解为通过主机内存的 GPU→CPU→GPU 路径的一段）。C2C 常用于描述需要借助主机内存/CPU 参与的传输，而不是直接 GPU-to-GPU 的专用互连。

**典型场景**：
GPU 与本地 NUMA 域（CPU）之间的连接（用于 GPU↔主机内存的数据移动）。
当两块 GPU 没有 NVLink 或直接 P2P 时，数据可能走 GPU→CPU→GPU（即 C2C + PCIe），这时拓扑会包含 C2C 段。

**带宽**：通常受 PCIe 主机带宽和 CPU 内存子系统限制，并可能带有额外开销（例如 Intel 的 P2P over PCIe 的开销宏 INTEL_P2P_OVERHEAD）。

**用途**：用于评估跨主机/跨 NUMA 的传输成本，以及 GPU 与 NIC 交互的路径（比如 PATH_P2C 描述“GPU到NIC 使用 C2C 到 CPU 再走 PCIe 到 NIC”）。

## LINK_PCI (4) — PCIe（PCI Express）链路

**含义**：表示 PCIe 拓扑上的链路（GPU ↔ PCIe switch/to PCI device/根复合体）。

**典型场景**：
GPU 节点与 PCIe 开关/桥或 NIC（作为 PCI device）之间的直接 PCIe 连接。
GPU 与主板上的 PCIe root complex 的连接段。

**带宽**：以 PCI_BW（代码中默认为 PCI Gen3 x16 的吞吐，示例常量 12.0 GB/s）为基准；根据实际 Gen/宽度或拓扑可调整。

**用途**：用于表示通过 PCIe 总线传输的段（比如 PATH_PIX、PATH_PXB、PATH_PHB 等不同粒度的 PCIe 路径都会用到 PCI 类型链路）。

## LINK_SYS (9) — 系统互连（System interconnect，如 QPI/UPI）

**含义**：表示主机 CPU 节点之间的系统互连（例如 Intel 的 QPI/UPI，或其他 CPU-to-CPU 互连），即 NUMA 节点间的互连。

**典型场景**：跨 socket/NUMA 的数据移动路径中，数据需要经过 CPU 间互连（例如 GPU 在 socket A，目标在 socket B，数据从 GPU→CPU(A)→跨 socket→CPU(B)→GPU）。

**带宽**：使用 QPI/UPI 类的带宽常量（代码定义了 BDW_QPI_BW、SKL_QPI_BW、SRP_QPI_BW、ERP_QPI_BW 等，具体取决于 CPU 平台）。

**用途**：用于评估跨 NUMA 节点的代价（PATH_SYS），这对选择是否跨 socket 做通信聚合或走本地通路非常重要。

## LINK_NET (10) — 网络（Network）链路

**含义**：表示通过外部网络（例如 100GbE/InfiniBand/RDMA 网络）进行的链路，即主机与远端主机之间的网络连接段。

**典型场景**：NIC ↔ 网络交换机/远端主机，或本机 NET 节点表示远端主机的对应端点。

**带宽**：以 NET_BW（默认例如 12.0 GB/s 表示 100 Gbit） 或实际网卡速率为准（可能根据插件/检测进行调整）。

**用途**：构建跨主机的拓扑路径（PATH_NET），评估远程通信开销，决定是否使用网卡直连（RDMA / GPUDirect）或需走主机内存中转。
补充说明（设计与映射）：

## 跳过的link类型
上面代码中有些常数被“跳过”（例如 2、5、6、7、8），那是为了让 link type 与 path type 的编号尽量对齐/兼容（历史或语义上的原因），因此某些数字保留给 path 类型或留作将来扩展。实际使用时，常见的 link type 就是上面列出的几种。

对应的 path type（PATH_*）与 link type 关系大致是：
- PATH_LOC ↔ LINK_LOC
- PATH_NVL / PATH_PORT ↔ LINK_NVL
- PATH_C2C ↔ LINK_C2C
- PATH_PIX / PATH_PXB / PATH_PHB 等 ↔ LINK_PCI（只是细分了 PCIe 穿越的层次）
- PATH_SYS ↔ LINK_SYS
- PATH_NET ↔ LINK_NET

## 举例说明

在 topo 中：ncclTopoNode 的 links[] 数组里每项是 ncclTopoLink，包含 type（上面枚举之一）、bw（带宽）、remNode（对端节点）。路径查找会遍历这些 link 来构建从某一 GPU 到另一端（GPU/NIC/NET）的最优路径及其带宽/延迟估计。

示例（便于理解）：

两块在同一主机并直接用 NVLink 互联的 GPU：它们之间的链路为 LINK_NVL，bw=ncclTopoNVLinkBw(cudaCap)。NCCL 会倾向用 NVLink 做 GPU-GPU 通信。
两块不连 NVLink、但在同一主机且能通过 PCIe 直接 Peer-to-Peer：路径会包含 LINK_PCI（GPU→PCIe switch→GPU），并可能组合 LINK_C2C 表示主机内存中转段，带宽使用 PCI_BW 并考虑额外开销。

GPU 发送要到远端主机的 GPU：路径会包含 LINK_PCI（GPU→NIC） + LINK_NET（网络段） + 远端的 LINK_PCI/NODE 段等。
