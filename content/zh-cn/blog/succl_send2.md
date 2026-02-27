---
title: "xCCL send recv 阻塞"
linkTitle: "xCCL send recv 阻塞"
date: 2026-02-03
description: >
    Debug xCCL send recv 阻塞
---


场景如下：

- 每个GPU一个进程
- 两个进程：rank 0, rank 1
- rank 0 调用send  然后调用计算kernel。
- rank 1 sleep 10 seconds，然后调用recv

预期结果：计算kernel不被阻塞
实际结果：计算kernel被阻塞，直到rank 1 recv接收到数据然后rank 0才开始做计算。


![alt text](/cs-note/image/image-1.png)

![alt text](/cs-note/image/image-2.png)


且当send的数据是512字节对齐的时候，也不会被阻塞。
![alt text](/cs-note/image/image-3.png)

分析：
`copyInternal`是 SDMA ， 而 `bcastInternal` 是CPDMA。根据测试情况，可以确定是SDMA导致的问题。


从硬件角度：

kernel launch，第一次launch这个kernel的时候，需要加载这个文件所在的所有的指令到GPU用到的就是SDMA。当第二次launch的时候，就不需要用SDMA了。

SDMA有两个EU，绑定着两个pipe，queue 和pipe的绑定关系是固定的，不会发生变化。有可能graph launch用的 sdma queue 和kernel launch时copy kernel用的sdma queue 绑定到了同一个pipe上。这里queue的概念就类似unsafe mode stream。

