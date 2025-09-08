```cpp
typedef struct ncclComm* ncclComm_t;

struct ncclComm {
  uint64_t startMagic;                             // 一个数字，用于校验数据
  struct ncclMemoryStack memPermanent, memScoped;
  // List of destructors to run when comm is destructed
  struct ncclDestructor* destructorHead;

  struct ncclCudaContext* context;
  struct ncclSharedResources* sharedRes;
  /* map to top parent ranks. */
  int* topParentRanks;
  int* topParentLocalRanks;
  struct ncclChannel channels[MAXCHANNELS];
  struct ncclPeerInfo* peerInfo;
  struct ncclTopoSystem* topo;
  struct ncclProxyConnector* gproxyConn;
  struct ncclIntruQueue<struct ncclCommCallback, &ncclCommCallback::next> legacyRegCleanupQueue;
  bool peerInfoValid;

  ncclNet_t* ncclNet;
  int netPluginIndex;
  int ncclNetVer;
  ncclNetDeviceType netDeviceType;
  ncclCollNet_t* ncclCollNet;
  void* bootstrap;
  // Bitmasks for ncclTransportP2pSetup
  uint64_t* connectSend;
  uint64_t* connectRecv;
  struct ncclTopoGraph graphs[NCCL_NUM_ALGORITHMS];
  int maxTreePattern;
  bool initAlgoChannels[NCCL_NUM_ALGORITHMS];
  bool runtimeConn; // if dynamic connection is supported
  bool directMode;
  int cuMemSupport;

  uint64_t magic; // Magic number for all network communication. Not a security key -- only goal is to detect mismatches.

  uint64_t commHash;
  int rank;    // my rank in the communicator
  int nRanks;  // number of GPUs in communicator
  int cudaDev; // my cuda device index
  int nvmlDev; // my nvml device index
  int compCap; // compute capability of the GPU
  int minCompCap, maxCompCap; // min/max compute capability in the communicator
  int64_t busId;   // my PCI bus ID in int format
  cpu_set_t cpuAffinity; // CPU affinity of the GPU
  int cudaArch; // matches __CUDA_ARCH__ of device

  int cpuArch;   // architecture - As defined in src/include/graph.h, e.g. x86/arm/ppc/mixed
  int cpuVendor; // vendor - As defined in src/include/graph.h

  int node;
  int nNodes;
  int localRank;
  int localRanks;
  int maxLocalRanks;
  int* rankToNode;
  int* rankToLocalRank;
  int* localRankToRank;
  // localRanks and localRanktoRank for all nodes
  struct ncclNodeRanks* nodeRanks;
  // MNNVL: Multi-Node NVLink
  int MNNVL; // true when MNNVL is available
  struct cliqueInfo clique; // Our MNNVL clique information
  int cliqueRank; // Our rank within the MNNVL clique

  bool checkPointers;
  bool dmaBufSupport;

  // Counter for tracking CUDA launches (P2P and collectives included)
  uint64_t opCount;
  // Collective operation counter
  uint64_t collOpCount;

  // Channels for collectives
  int nChannels; // connection nChannels
  int collChannels; // enqueue nChannels
  int nvlsChannels; // enqueue nChannels
  // all nvls heads stored to check if we can splitShare
  int nvlsHeads[MAXCHANNELS];
  // Channels (per peer) for p2p
  int p2pnChannels;
  int p2pnChannelsPerPeer;

  // Should this comm allocate LL buffers for network P2P connections?
  bool allocP2pNetLLBuffers;

  // Buffer sizes
  int buffSizes[NCCL_NUM_PROTOCOLS];
  int p2pChunkSize;
  int nvlsChunkSize;

  // Algorithm/Protocols thresholds
  ssize_t threadThresholds[NCCL_NUM_ALGORITHMS][NCCL_NUM_PROTOCOLS];
  float latencies[NCCL_NUM_FUNCTIONS][NCCL_NUM_ALGORITHMS][NCCL_NUM_PROTOCOLS];
  float bandwidths[NCCL_NUM_FUNCTIONS][NCCL_NUM_ALGORITHMS][NCCL_NUM_PROTOCOLS];
  int maxThreads[NCCL_NUM_ALGORITHMS][NCCL_NUM_PROTOCOLS];

  /* This attribute can indicate the states of communicators and return code of
   * asynchronous NCCL operations. */
  ncclResult_t asyncResult;

  // Flag to ask NCCL kernels to abort
  uint32_t* abortFlag;
  uint32_t* abortFlagDev;
  int* abortFlagRefCount;
  uint32_t* childAbortFlag;
  uint32_t* childAbortFlagDev;
  uint32_t destroyFlag;

  // Device side of the communicator (for cudaFree's)
  struct ncclDevComm* devComm; // actually = &ncclDevCommAndChannels::comm
  struct ncclSymDevComm symDevComm;

  uint32_t workArgsBytes; // max size of kernel args
  uint32_t workFifoBytes; // size of workFifoBuf, power of 2
  void* workFifoBuf;
  void* workFifoBufDev;
  void* workFifoBufGdrHandle;

  // Monotonic number of bytes (mod 1<<32) sent to fifo.
  uint32_t workFifoProduced;
  uint32_t workFifoProducedLastRecorded;
  uint32_t workFifoConsumed;

  // Intra-process sync
  struct ncclComm* intraComm0; // leader of intra-process comms (self possible)
  struct ncclComm* intraNext; // next of intra-process comms, intraComm0 is head
  int intraRank;
  int intraRanks;
  uint32_t intraBarrierPhase;
  char intraPad1[64 - sizeof(uint64_t)];
  uint64_t intraBarrierCounter; // only used if this is intraComm0
  char intraPad2[64 - sizeof(uint64_t)];
  uint64_t intraBarrierGate; // only used if this is intraComm0

  struct ncclProxyState* proxyState;
  int proxyRefCountOld; /* store proxy post-atomic-sub refcount */
  // Whether this communicator uses collNet
  bool isOneRPN;
  uint8_t collNetSupportMatrix[4/*sum,prod,max,min*/][ncclNumTypes];
  int* collNetHeads;
  int collNetHeadsNum;
  int* collNetDenseToUserRank;
  int* collNetUserToDenseRank;
  /* sharable collNet proxy progress resource. */
  struct ncclCollNetSharedRes* collNetSharedRes;

  // NVLink SHARP (NVLS) support
  int nvlsSupport;
  int nvlsRegSupport;
  /* sharable NVLS resource. */
  struct ncclNvlsSharedRes* nvlsResources;

  // pools backed by comm->memPermanent
  struct ncclMemoryPool memPool_ncclTaskColl;
  struct ncclMemoryPool memPool_ncclTaskP2p;
  struct ncclMemoryPool memPool_ncclProxyOp;
  struct ncclMemoryPool memPool_ncclKernelPlan;

  // Next comm in this thread's active ncclGroup[Start|End](). Holds "0x1" when
  // this comm is not yet in a group.
  struct ncclComm* groupNext[ncclGroupTaskTypeNum];
  // Subset of those in groupNext list. Holds 0x1 if not needing preconnect.
  struct ncclComm* preconnectNext;
  int localPersistentRefs; // number of persistent plan-lists capturing this comm
  struct P2pSchedulePair { int sendRank; int recvRank; } *p2pSchedule;

  struct ncclKernelPlanner planner;

  cudaMemPool_t memPool;
  // Queue of events and associated callbacks for cleaning up asynchronous work.
  // Using this is preferable to using CUDA host callbacks because host callbacks
  // won't allow the work following the callback to run until the callback completes,
  // which comes at expense to perf.
  struct ncclIntruQueue<struct ncclCommEventCallback, &ncclCommEventCallback::next> eventCallbackQueue;

  // user-created reduction ops
  int userRedOpCapacity, userRedOpFreeHead;
  ncclUserRedOp *userRedOps;

  // Queue of things for the main thread to do
  int reclaimSteps;
  struct ncclIntruQueueMpsc<struct ncclCommCallback, &ncclCommCallback::next> callbackQueue;

  ncclConfig_t config;
  // initState is to more conveniently reclaim resources when errors happen.
  ncclResult_t initState;
  // flag to indicate if ncclCommFinalize() is called
  bool finalizeCalled;
  // shared structures for finalization
  int finalizeRankCnt;
  // group job to support multi-thread FT
  struct ncclGroupJob *groupJob;

  // Flag indicating if this communicator shares resources with parent or children
  bool shareResources;

  // Tuning plugin
  int tunerPluginLoaded;
  ncclTuner_t* tuner;
  void *tunerContext;

  // Profiler plugin
  void* profilerContext;
  uint64_t seqNumber[NCCL_NUM_FUNCTIONS];
  struct ncclProfilerProxy profiler;

  // buffer registration cache
  struct ncclRegCache regCache;
  int isAllNvlink;
  bool isAllDirectP2p;
  int symmetricSupport;
  bool useNetPXN;
  bool useGdr;
  int splitCount;
  // symmetric buffer
  uint8_t* baseUCSymPtr;
  uint8_t* baseMCSymPtr;
  size_t baseStride;
  size_t symAllocHead;
  CUmemGenericAllocationHandle symMCHandle;
  struct ncclIntruQueue<struct ncclSymRegTask, &ncclSymRegTask::next> symRegTaskQueue;
  uint64_t endMagic;
};


```
