

创建proxy线程
```cpp
static ncclResult_t ncclProxyProgressCreate(struct ncclProxyState* proxyState) {
  struct ncclProxyProgressState* state = &proxyState->progressState;
  if (!state->thread) {
    PTHREADCHECK(pthread_create(&state->thread, NULL, ncclProxyProgress, proxyState), "pthread_create");
    ncclSetThreadName(state->thread, "NCCL Progress%2d", proxyState->tpLocalnRanks);
  }
  return ncclSuccess;
}
```

```cpp
void* ncclProxyProgress(void *proxyState_) {
  struct ncclProxyState* proxyState = (struct ncclProxyState*)proxyState_;
  if (setProxyThreadContext(proxyState)) {
    INFO(NCCL_INIT, "[Proxy Progress] Set CUDA context on device %d", proxyState->cudaDev);
  } else if (cudaSetDevice(proxyState->cudaDev) != cudaSuccess) {
    WARN("[Proxy Progress] Failed to set CUDA device %d", proxyState->cudaDev);
  }
  // if (CPU_COUNT(&comm->cpuAffinity)) sched_setaffinity(0, sizeof(cpu_set_t), &comm->cpuAffinity);

  INFO(NCCL_INIT, "[Proxy Progress] Device %d CPU core %d", proxyState->cudaDev, sched_getcpu());

  struct ncclProxyProgressState* state = &proxyState->progressState;
  state->nextOps = -1;
  const int sig = ncclParamProxyDumpSignal();
  if (sig != -1) signal(sig, ncclDumpProxyState);
  ncclLastProxyState = state;
  char threadName[NCCL_THREAD_NAMELEN];
  snprintf(threadName, NCCL_THREAD_NAMELEN, "NCCL Progress%2d", proxyState->cudaDev);
  nvtxNameOsThreadA(syscall(SYS_gettid), threadName);

  int lastIdle = 0;
  /* Too frequent call of ncclProxyGetPostedOps() will result in perf regression for small message
   * communication. proxyOpAppendCounter is a counter that helps us decide if we need to append proxy ops.
   * After each progress, proxyOpAppendCounter will increase by 1 and compare with environment variable
   * ncclParamProgressAppendOpFreq(). If they are equal, we will append proxy ops. This will decrease the
   * frequency of calling ncclProxyGetPostedOps() and reduce the perf impact. */
  int proxyOpAppendCounter = 0;
  do {
    int idle = 1;
    ncclResult_t ret = progressOps(proxyState, state, state->active, &idle);
    if (ret != ncclSuccess) {
      __atomic_store_n(&proxyState->asyncResult, ret, __ATOMIC_RELEASE);
      INFO(NCCL_ALL,"%s:%d -> %d [Progress Thread]", __FILE__, __LINE__, ret);
      break;
    }
    void* eHandle;
    ncclProfilerStartProxyCtrlEvent(proxyState->profilerContext, &eHandle);
    if (lastIdle == 0 && idle == 1) ncclProfilerRecordProxyCtrlEventState(eHandle, 0, ncclProfilerProxyCtrlIdle);
    if (lastIdle == 1 && idle == 0) ncclProfilerRecordProxyCtrlEventState(eHandle, 0, ncclProfilerProxyCtrlActive);
    ncclProfilerStopProxyCtrlEvent(eHandle);
    if (idle || !state->active || (++proxyOpAppendCounter == ncclParamProgressAppendOpFreq())) {
      int added = 0;
      proxyOpAppendCounter = 0;
      TIME_START(3);
      ret = ncclProxyGetPostedOps(proxyState, &added);
      if (added) { TIME_STOP(3); } else { TIME_CANCEL(3); }
      if (ret != ncclSuccess) {
        __atomic_store_n(&proxyState->asyncResult, ret, __ATOMIC_RELEASE);
        INFO(NCCL_ALL,"%s:%d -> %d [Progress Thread]", __FILE__, __LINE__, ret);
      }
      if (added == 0) {
        sched_yield(); // No request progressed. Let others run.
      }
    }
    lastIdle = idle;
  } while ((state->stop == 0 || (state->stop == 1 && state->active)) && __atomic_load_n(proxyState->abortFlag, __ATOMIC_ACQUIRE) == 0);
  return NULL;
}
```
