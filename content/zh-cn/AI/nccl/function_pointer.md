---
title: NCCL中的函数指针
weight: 2
date: 2025-08-28
description:
  NCCL中很多地方使用了函数指针。阅读含有大量函数指针的代码，通常比较难以理解，这里介绍一下，NCCL中的函数指针。
---


```cpp
struct ncclTransport {
  const char name[8];
  ncclResult_t (*canConnect)(int*, struct ncclComm* comm, struct ncclTopoGraph* graph, struct ncclPeerInfo*, struct ncclPeerInfo*);
  struct ncclTransportComm send;
  struct ncclTransportComm recv;
};
```

