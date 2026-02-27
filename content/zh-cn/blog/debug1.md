---
title: NCCL hang debug 
linkTitle: NCCL hang debug 
date: 2025-10-24
description: >
    RDMA、NCCL、pytorch、megatron
---

# 问题描述

升级megatron-lm和megatron-lm-br(公司插件)到最新版本，在做启动时候遇到了**程序卡住hang**的问题。

通过定位发现是NCCL hang住了。



## 测试步骤

从NCCL角度看，构造简化的测试用例：
测试环境：2机16卡，IB网卡
1. 创建 通信组comm，
2. 创建子通信组comm split，生成子comm，
3. 通过系统调用fork方式，产生子进程。
4. 用父comm进行all reduce操作。（这一步hang了）

## 问题分析


通过日志会发现，NCCL是hang在了RDMA 通信，两个rank之间进行rdma 通信，会发现pull不到成功的CQE。

之所以pull不到成功的CQE， 一个机器上的rank的ISend 等不到post fifo的地址数据，尽管对端已经post fifo，并且拿到了CQE。

连续几天对比NCCL日志和RDMA的日志，均为发现有什么异常，但之后才注意到测试步骤中，是先创建了通信组，然后才fork的进程。

NVIDIA的官方的文档中，在CUDA 12 中还有描述，CUDA不支持 在做了初始化之后进行fork操作。

> https://docs.nvidia.com/cuda/archive/12.2.1/cuda-toolkit-release-notes/index.html?highlight=fork#general-cuda
> 12.2.1 cuda Release Notes:
> The fork() system call is not fully supported yet when attempting to share GPU-accessible memory between parent and child processes.




- 根本原因：当使用了CUDA初始化之后，再进行fork产生子进程，会造成未定义行为。
- 解决办法：在SCLL初始化之前创建子进程，或者采取spawn的方式创建子进程。


## 总结

**DEBUG过程，一定要先了解清楚硬件信息（型号、拓扑），测试步骤等信息，然后才进行日志分析。**

技术总结：

最佳实践：于使用CUDA、NCCL、RDMA的程序来说，在进行了初始化操作后，不应该再fork出子进程。

- 对于CUDA，NVIDIA在早期文档中明确说明不支持fork，但是慢慢的已经开始支持部分了，但是还没有完全支持，但是很多文档删除了不支持fork的说明，还有一些文旦给保留着。
除非有必要，不要做这样的编程。
如NVIDIA Magnum IO GPUDirect Storage
The fork system call should not be used after the library is initialized. The behavior of the APIs
after the fork system call is undefined in the child process

- 对于NCCL，fork的子进程和父进程几乎拥有一样的资源，比如文件描述符，socket链接、rank编号等，NCCL也使用了CUDA，由于这些资源一样. socket一样，也会对通信进行干扰。

- 对于RDMA，RDMA本身ibv_fork_init提供了这个接口，以防止fork出现未定义行为，但是在子进程中还是需要重新创建qp等一些列的操作比较好，这也不是常规的使用方式。


fork和spawn的对比

父进程调用fork()时，操作系统会创建一个与父进程几乎完全相同的子进程。具体来说：  

子进程会复制父进程的内存空间（包括代码段、数据段、堆、栈）、文件描述符（打开的文件、网络连接等）、进程上下文（寄存器、程序计数器等）。  

为了提高效率，操作系统采用写时复制（Copy-On-Write, COW） 机制：初始时子进程并不实际复制父进程的内存数据，而是与父进程共享同一块内存；只有当子进程或父进程修改数据时，才会触发内存的复制（仅复制被修改的页）。

子进程从fork()调用的返回处开始执行（与父进程的执行逻辑分离）。

因为RDMA的资源是pin住内存的？RDMA如何处理？ 调用ibv_fork_init后，告知 libibverbs 后续可能发生fork。在fork发生时，自动父进程资源为无效，自己进程重新初始化。

当父进程通过spawn创建子进程时，操作系统会启动一个全新的 Python 解释器进程，而非复制父进程。具体流程：
父进程将需要传递给子进程的参数（如目标函数、初始化数据）通过序列化（如pickle）传递给子进程；
子进程启动后，会重新初始化 Python 解释器环境，重新导入父进程中用到的模块，并反序列化父进程传递的参数；
子进程仅继承父进程的必要资源（如标准输入输出、环境变量），其他资源（如内存数据、文件描述符）不会自动继承。



## 尚未解决的问题：

- 为什么RDMA已经调用了ibv_fork_init，但是RDMA还是出问题了？还没有一个好的猜测。




## 参考链接：

- https://blog.csdn.net/handsomeboysk/article/details/149935280
- https://forums.developer.nvidia.com/t/cuda8-0-bug-child-process-forked-after-cuinit-get-cuda-error-not-initialized-on-cuinit/45764
- https://github.com/NVIDIA/cuda-python/issues/1136
- https://blog.csdn.net/YNNAD1997/article/details/113829532



- https://docs.nvidia.com/cuda/archive/11.4.0/gds/pdf/best-practices-guide.pdf
> The fork system call should not be used after the library is initialized. The behavior of the APIs
after the fork system call is undefined in the child process.


- https://docs.nvidia.com/cuda/archive/12.2.1/cuda-toolkit-release-notes/index.html?highlight=fork#general-cuda
> 12.2.1 cuda Release Notes:
> The fork() system call is not fully supported yet when attempting to share GPU-accessible memory between parent and child processes.

- https://www.pascal-man.com/gpu-cuda/CUDA_C_Programming_Guide.pdf
> K.2.5.2. Using fork() with Managed Memory
> The Unified Memory system does not allow sharing of managed memory pointers
> between processes. It will not correctly manage memory handles that have been
> duplicated via a fork() operation. Results will be undefined if either the child or parent
> accesses managed data following a fork().
> It is safe, however, to fork() a child process that then immediately exits via an exec()
> call, because the child drops the memory handles and the parent becomes the sole owner
> once again. It is not safe for the parent to exit and leave the child to access the handles.


- https://docs.pytorch.org/docs/stable/notes/multiprocessing.html
> Poison fork in multiprocessing
> When using multiprocessing with accelerators, a known issue called “poison fork” may occur. This happens when the accelerator’s runtime is not fork safe and is initialized before a process forks, leading to runtime errors in child processes.
> 
> To prevent such errors:
> Avoid initializing the accelerator in the main process before forking child processes.
> 
> Use an alternative process start methods, such as spawn or forkserver, which ensures a clean initialization of each process.
> 
> CUDA in multiprocessing
> The CUDA runtime has the limitation described in Poison fork in multiprocessing when using the fork start method; either the spawn or forkserver start method are required to use CUDA in subprocesses.
> 
> Note
> 
> The start method can be set via either creating a context with multiprocessing.get_context(...) or directly using multiprocessing.set_start_method(...).
> 
> Unlike CPU tensors, the sending process is required to keep the original tensor as long as the receiving process retains a copy of the tensor. It is implemented under the hood but requires users to follow the best practices for the program to run correctly. For example, the sending process must stay alive as long as the consumer process has references to the tensor, and the refcounting can not save you if the consumer process exits abnormally via a fatal signal. See this section.
> 
> See also: Use nn.parallel.DistributedDataParallel instead of multiprocessing or nn.DataParallel
