---
title: NCCL中的同步机制
weight: 3
date: 2025-09-05
description:
  NCCL中的例子
---


```cpp
template <int DirectRecv, int DirectSend, int Recv, int Send, int Src, int Dst>
__device__ __forceinline__ void waitPeer(intptr_t srcIx, intptr_t dstIx, int offset, int nelts) {
const bool isSendNotRecv = (Send && Recv) ? (flags & RoleWaitSend) : Send;
// Yes, for some template arguments this code will be unreachable.  That's fine.
// coverity[dead_error_line]
if ((flags & (Recv * RoleWaitRecv)) || (flags & (Send * RoleWaitSend))) {
    int spins = 0;
    while (connStepCache + (isSendNotRecv ? NCCL_STEPS : 0) < step + StepPerSlice) {
    connStepCache = loadStepValue(connStepPtr);
    if (checkAbort(flags, Aborted, spins)) break;
    //if (spins == 0) printf("r=%d b=%d t=%d SPUN OUT got=%d want=%d\n", ncclShmem.comm.rank, blockIdx.x, threadIdx.x, int(connStepCache + (isSendNotRecv ? NCCL_STEPS : 0)), int(step+StepPerSlice));
    }
}

if (flags & (Recv*RoleWaitRecv | Send*RoleWaitSend)) {
    if ((flags & ConnFifoEnabled) && (flags & (Send * RoleWaitSend)))
    connFifo[step%NCCL_STEPS].size = nelts*sizeof(T);

    void **ptrs = isSendNotRecv ? (ncclShmem.groups[group].dsts + Dst)
                                : (ncclShmem.groups[group].srcs + Src);
    if ((flags & NetRegMode) && ((!isSendNotRecv && DirectRecv) || (isSendNotRecv && DirectSend))) {
    if (P2p) {
        ptrs[index] = NULL;
    } else {
        if (isSendNotRecv) {
        if (!Recv)
            ptrs[index] = NULL;
        else
            ptrs[index] = (T*)ncclShmem.groups[group].userOutput + dstIx + offset;
        } else {
        ptrs[index] = (T*)ncclShmem.groups[group].userOutput + srcIx + offset;
        }
    }
    } else if ((flags & ConnFifoEnabled) && connFifo[step%NCCL_STEPS].mode == NCCL_MODE_OFFSET) {
    ptrs[index] = connEltsFifo + loadInt(&connFifo[step%NCCL_STEPS].offset)/sizeof(T);
    } else if (isSendNotRecv && DirectSend) {
    if (flags & DirectWrite) {
        ptrs[index] = directBuff + dstIx + offset;
    } else if (flags & DirectRead) {  // empty send
        ptrs[index] = nullptr;
    } else {
        ptrs[index] = connEltsFifo + (step%NCCL_STEPS)*connStepSize;
    }
    } else if (!isSendNotRecv && DirectRecv) {
    if (flags & DirectRead) {
        ptrs[index] = directBuff + srcIx + offset;
    } else if (flags & DirectWrite) {
        ptrs[index] = directBuff + dstIx + offset;  // send to next from my output buffer
    } else {
        ptrs[index] = connEltsFifo + (step%NCCL_STEPS)*connStepSize;
    }
    }
    else {
    // Yes, for some template arguments this code will be unreachable.  That's fine.
    // coverity[dead_error_line]
    ptrs[index] = connEltsFifo + (step%NCCL_STEPS)*connStepSize;
    }
    if (flags & NetDeviceUnpack) {
    ncclNetDeviceIncrementHead(group, index);
    }
    step += StepPerSlice;
}
}

template<int Recv, int Send>
inline __device__ void postPeer(bool dataStored) {
    if (flags & (Recv*RolePostRecv | Send*RolePostSend)) {
        step += StepPerSlice;
        if (Send && (flags & RolePostSend) && (dataStored||(flags&ConnFifoEnabled))) {
            fence_acq_rel_sys();
        }
        st_relaxed_sys_global(connStepPtr, step);
    }
}
```
