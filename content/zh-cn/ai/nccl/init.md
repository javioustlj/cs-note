
# ncclCommInitAll

## ncclGetUniqueId

### ncclInit

#### initEnv

#### initGdrCopy

#### bootstrapNetInit

#### initNvtxRegisteredEnums

### bootstrapGetUniqueId

64位的随机数 + bootstrap网络接口地址

#### bootstrapCreateRoot

- ncclSocketInit 创建TCP socket
- 

## ncclCommInitRankDev

```cpp
  NCCLCHECKGOTO(ncclGroupStartInternal(), ret, fail);
  for (int i=0; i<ndev; i++) {
    // Ignore return codes .. we need to call ncclGroupEnd to clean up anyway
    int dev = devlist ? devlist[i] : i;
    CUDACHECKGOTO(cudaSetDevice(dev), ret, fail);
    ncclCommInitRankDev(comms+i, ndev,1, &uniqueId, i, dev, &config, __func__);
  }
```
