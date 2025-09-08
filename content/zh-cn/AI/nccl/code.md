---
title: NCCL中有意思的代码
weight: 100
date: 2025-08-29
description:
  NCCL中看起来不合理，但是巧妙的代码
---

释放空指针，这种用法通常用于**初始化 CUDA 上下文**或**验证 CUDA 环境**是否正常工作（比如检查驱动是否加载、设备是否可用等）。
```cpp
  // Make sure the CUDA runtime is initialized.
  CUDACHECKGOTO(cudaFree(NULL), res, fail);
```


# 模板

```cpp
template<typename T, T *T::*next>
struct ncclIntruQueue {
  T *head, *tail;
};

__thread struct ncclIntruQueue<struct ncclAsyncJob, &ncclAsyncJob::next> ncclAsyncJobs;
```

