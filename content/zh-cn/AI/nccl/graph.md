---
title: link
linktitle: link
weight: 3
date: 2025-12-11
description:
  NCCL中的Graph
---

在学习算法过程中，图一般用二维数组表示或者邻接矩阵表示。但是在NCCL中，自己设计了一种针对特定场景的图的表示的方法。

`struct ncclTopoGraph` 既不是传统的二维数组表示，也不是邻接矩阵表示，而是一种专门为NCCL集体操作通信模式设计的特殊图表示结构 [1](#1-0) 。

## 图表示方式分析

### 扁平化的通道-节点表示

`ncclTopoGraph` 使用两个一维数组来表示图结构：

- **`intra[]`**: 存储每个通道内GPU的通信顺序，按通道扁平化存储 [2](#1-1) 
- **`inter[]`**: 存储每个通道的NIC连接（发送和接收），每通道2个NIC ID [3](#1-2) 

这种布局可以理解为：
```
Channel 0: intra[0..ngpus-1], inter[0..1]
Channel 1: intra[ngpus..2*ngpus-1], inter[2..3]
Channel 2: intra[2*ngpus..3*ngpus-1], inter[4..5]
...
```

### 与传统图表示的区别

| 表示方式 | 存储结构 | 适用场景 | NCCL中的使用 |
|---------|---------|---------|-------------|
| 邻接矩阵 | `matrix[i][j]` = 连接关系 | 稠密图 | 不适用 |
| 邻接表 | `list[i]` = 邻接节点 | 稀疏图 | 部分相关 |
| **NCCL图** | **通道+序列** | **集体通信模式** | **专门设计** |

### 实际使用示例

在图搜索过程中，这种表示方式用于构建特定的通信模式<cite repo="NVIDIA/nccl" path="src/graph/search.cc" start="596" end="596">：

```c
// 记录当前通道的第step个GPU
graph->intra[graph->nChannels*ngpus+step] = gpu->gpu.rank;
```

在连接建立时，直接按通道索引访问GPU序列<cite repo="NVIDIA/nccl" path="src/graph/connect.cc" start="41" end="43">：

```c
int* ringIntra = graphs[NCCL_ALGO_RING]->intra+c*localRanks;
int* treeIntra = graphs[NCCL_ALGO_TREE]->intra+c*localRanks;
```

## 设计优势

1. **内存局部性**：同一通道的节点连续存储，缓存友好
2. **访问效率**：O(1)时间访问任意通道的任意位置
3. **模式特化**：针对ring、tree等固定模式优化
4. **并行友好**：不同通道可以独立处理

## Notes

- 这种表示是NCCL为高性能集体通信专门设计的，不是通用图数据结构
- 它牺牲了通用性，换取了在特定通信模式下的极致性能
- 图的"边"信息隐含在GPU序列的相邻关系中，而不是显式存储

Wiki pages you might want to explore:
- [Path Computation and Graph Search](/wiki/NVIDIA/nccl#3.2)

Wiki pages you might want to explore:
- [Topology System (NVIDIA/nccl)](/wiki/NVIDIA/nccl#3)
- [Communication Pipeline (NVIDIA/nccl)](/wiki/NVIDIA/nccl#4)

### Citations

**File:** src/include/graph.h (L101-120)
```text
struct ncclTopoGraph {
  // Input / output
  int id; // ring : 0, tree : 1, collnet : 2, nvls : 3, collnetDirect : 4
  int pattern;
  int crossNic;
  int collNet;
  int minChannels;
  int maxChannels;
  // Output
  int nChannels;
  float bwIntra;
  float bwInter;
  float latencyInter;
  int typeIntra;
  int typeInter;
  int sameChannels;
  int nHops;
  int intra[MAXCHANNELS*NCCL_TOPO_MAX_NODES];
  int64_t inter[MAXCHANNELS*2];
};
```

