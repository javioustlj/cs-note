---
title: BootStrap
weight: 2
date: 2025-08-28
description:
  BootStrap 网络
---


```cpp
struct extInfo {
  int rank;                                  // rank of the process reaching out
  int nranks;                                // total number of ranks
  int iroot;                                 // current root index
  int nroots;                                // total number of roots
  union ncclSocketAddress listenRootAddress; // address of my listenSocket for the root
  union ringConnectInfo connectInfo;
};
```

rank 0 会启动一个线程 "NCCL BootstrapR"



# bootstrapNetInit

作用：找到可用的网络接口，并将它的信息保存下来。

- 如果配置了`NCCL_COMM_ID`，则使用和它在同一网段的网口。
- `ib` 网络接口
- 普通网口
- docker 网口
- loopback 网口


