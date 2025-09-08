---
title: concepts
weight: 2
date: 2025-02-14
description:
  Learn concepts in NCCL.
---


## concepts

- node: 一个服务器
- CUDA device：一个GPU
- rank: GPU的编号
- communicator: 这是一个抽象的概念，用于管理一组GPU之间的通信。
- IB GDA


# temp

full-blown

multi-GPU parallelization model:

- single-threaded control of all GPUs
- multi-threaded, for example, using one thread per GPU
- multi-process, for example, MPI

SHARP 

- NVLink Sharp
- IB Sharp

Algorithm

- NVLS
- IB Sharp
