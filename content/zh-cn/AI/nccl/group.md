---
title: Group function
linktitle: Group function
weight: 3
date: 2026-02-04
description:
  NCCL中的Group Function 
---

# 实现

```cpp
// NCCL中的group操作时针对的一个CPU thread
__thread int ncclGroupDepth = 0; // depth of ncclGroupStart nesting
```

```cpp
// groupStart 即将ncclGroupDepth加1操作
inline ncclResult_t ncclGroupStartInternal() {
  ncclGroupDepth++;
  return ncclSuccess;
}
```

真正的操作是放在了`GroupEnd`这个函数里面

```cpp
struct ncclGroupJob {
  struct ncclAsyncJob base;
  int groupRefCount;
  bool nonBlockingInit;
  bool joined;
  struct ncclComm *groupCommHead[ncclGroupTaskTypeNum];
  struct ncclComm *groupCommPreconnectHead;
  ncclResult_t groupError;
  bool abortFlag;
  struct ncclIntruQueue<struct ncclAsyncJob, &ncclAsyncJob::next> asyncJobs;
};
```


ncclGroupCommPreconnectHead指向了一个列表头，这个列表是一个comm组，新的comm组都放到列表头。

```cpp
// 把comm
__thread struct ncclComm* ncclGroupCommPreconnectHead = nullptr;
// Add comm to this thread's group needing preconnect
inline void ncclGroupCommPreconnect(struct ncclComm* comm) {
  if (comm->preconnectNext == reinterpret_cast<struct ncclComm*>(0x1)) {
    comm->preconnectNext = ncclGroupCommPreconnectHead;
    ncclGroupCommPreconnectHead = comm;
  }
}
```

# comm 管理

```cpp
struct ncclComm {
  // Intra-process sync
  // 指向当前进程内所有 communicator 的“头”/领导者（可能是自己）。每个 communicator 的 intraComm0 都指向同一个头 communicator
  struct ncclComm* intraComm0; // leader of intra-process comms (self possible)
  struct ncclComm* intraNext; // next of intra-process comms, intraComm0 is head
};

```
