---
title: socket
weight: 10
date: 2025-08-28
description:
  nccl 中用到的socket
---

```cpp
ncclResult_t ncclSocketGetAddrFromString(union ncclSocketAddress* ua, const char* ip_port_pair);
ncclResult_t ncclFindInterfaceMatchSubnet(char* ifName, union ncclSocketAddress* localAddr,
                                          union ncclSocketAddress* remoteAddr, int ifNameMaxSize, int* found);
ncclResult_t ncclFindInterfaces(char* ifNames, union ncclSocketAddress *ifAddrs, int ifNameMaxSize, int maxIfs,
                                int* nIfs);

// Initialize a socket
ncclResult_t ncclSocketInit(struct ncclSocket* sock, const union ncclSocketAddress* addr = NULL, uint64_t magic = NCCL_SOCKET_MAGIC, enum ncclSocketType type = ncclSocketTypeUnknown, volatile uint32_t* abortFlag = NULL, int asyncFlag = 0, int customRetry = 0);
// Create a listening socket. sock->addr can be pre-filled with IP & port info. sock->fd is set after a successful call
ncclResult_t ncclSocketListen(struct ncclSocket* sock);
ncclResult_t ncclSocketGetAddr(struct ncclSocket* sock, union ncclSocketAddress* addr);
// Connect to sock->addr. sock->fd is set after a successful call.
ncclResult_t ncclSocketConnect(struct ncclSocket* sock);
// Return socket connection state.
ncclResult_t ncclSocketReady(struct ncclSocket* sock, int *running);
// Accept an incoming connection from listenSock->fd and keep the file descriptor in sock->fd, with the remote side IP/port in sock->addr.
ncclResult_t ncclSocketAccept(struct ncclSocket* sock, struct ncclSocket* ulistenSock);
ncclResult_t ncclSocketGetFd(struct ncclSocket* sock, int* fd);
ncclResult_t ncclSocketSetFd(int fd, struct ncclSocket* sock);

ncclResult_t ncclSocketProgress(int op, struct ncclSocket* sock, void* ptr, int size, int* offset, int* closed = NULL);
ncclResult_t ncclSocketWait(int op, struct ncclSocket* sock, void* ptr, int size, int* offset);
ncclResult_t ncclSocketSend(struct ncclSocket* sock, void* ptr, int size);
ncclResult_t ncclSocketRecv(struct ncclSocket* sock, void* ptr, int size);
ncclResult_t ncclSocketSendRecv(struct ncclSocket* sendSock, void* sendPtr, int sendSize, struct ncclSocket* recvSock, void* recvPtr, int recvSize);
ncclResult_t ncclSocketTryRecv(struct ncclSocket* sock, void* ptr, int size, int* closed, bool blocking);
ncclResult_t ncclSocketShutdown(struct ncclSocket* sock, int how);
ncclResult_t ncclSocketClose(struct ncclSocket* sock, bool wait = false);
```
