---
title: "xCCL Debug"
linkTitle: "xCCL Debug"
date: 2026-02-04
description: >
    Debug xCCL
---


## UMD

- export BR_UMD_TRACE=besu, 查看附近API报错的日志
- export BR_UMD_TRACE_LEVEL=1, 查看UMD的日志

- export SUPA_STREAM_NON_BLOCKING_UNSAFE=1, 可以强制把所有stream都变成unsafe，stream的queue不复用
- export BR_UMD_DEBUG_FORCE_ONE_SUBMIT_QUEUE=1 可以检查每个stream是否只用了一个queue
- export BR_UMD_TRACE_PACKET=1 抓下command packet log我们来分析下每个stream都发了啥任务占用了几个queue
