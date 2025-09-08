---
title: 一次 Docker 镜像迁移引发的 "水土不服" 事件
linkTitle: docker 加载驱动
date: 2025-04-16
description: >
    docker镜像迁移导致的环境不一致
---

奇怪问题之为什么docker容器内的环境并不一致


# 一次 Docker 镜像迁移引发的 "水土不服" 事件


## 问题的由来

上周帮同事迁移一个 Docker 环境时，遇到了件怪事：同一个镜像在物理机 A 上运行得好好的，搬到物理机 B 后就开始闹脾气 —— 应用启动失败、依赖库报错等问题。

迁移过后去到了一个编译问题：

```sh
Traceback (most recent call last):
  File "/workspace/br_pytorch/script/gen_pybind_stub.py", line 14, in <module>
    import pybind_supa
ImportError: /usr/local/birensupa/base/latest/supa/lib/libsupa-runtime.so.1: undefined symbol: besuMemcpyBroadcastAsyncWithGpuId, version LIBBESU_1.0
make[2]: *** [torch_br/csrc/pybind/CMakeFiles/pyi.dir/build.make:70: torch_br/csrc/pybind/CMakeFiles/pyi] Error 1
make[1]: *** [CMakeFiles/Makefile2:1302: torch_br/csrc/pybind/CMakeFiles/pyi.dir/all] Error 2
make[1]: *** Waiting for unfinished jobs....
```

首先检查一下libsupa-runtime.so.1的依赖项：

```sh
$ ldd -r  /usr/local/birensupa/base/latest/supa/lib/libsupa-runtime.so.1
        linux-vdso.so.1 (0x00007ffdeaf7b000)
        libbesu.so.1 => /usr/lib/libbesu.so.1 (0x00007fee52800000)
        libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fee525d4000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fee52c90000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fee52c70000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fee523ab000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fee52e25000)
        libbebr.so.1 => /usr/lib/libbebr.so.1 (0x00007fee51e00000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fee52c69000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fee52c64000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fee52c5f000)
undefined symbol: besuMemcpyBroadcastAsyncWithGpuId, version LIBBESU_1.0        (/usr/local/birensupa/base/latest/supa/lib/libsupa-runtime.so.1)
undefined symbol: besuMemcpyReduceAsyncWithGpuId, version LIBBESU_1.0   (/usr/local/birensupa/base/latest/supa/lib/libsupa-runtime.so.1)
```

从 `ldd -r` 输出结果来看，libsupa-runtime.so.1 库引用了 besuMemcpyBroadcastAsyncWithGpuId 和 besuMemcpyReduceAsyncWithGpuId 这两个符号，但这些符号在运行时找不到，并且版本要求为 LIBBESU_1.0。 可以看出LIBBESU_1.0的路径为：/usr/lib/libbesu.so.1。

看一下besu库的基础信息：

```sh
$ ls -al /usr/lib/libbesu.so.1
lrwxrwxrwx 1 root root 16 Mar 28 13:49 /usr/lib/libbesu.so.1 -> libbesu.so.1.6.0
```

可以看出它的版本是1.6.0


在物理机器A中的镜像中查看一下：

```sh
$ ls -al /usr/lib/libbesu.so.1
lrwxrwxrwx 1 root root 16 Mar 28 13:49 /usr/lib/libbesu.so.1 -> libbesu.so.1.7.0
```

结果却发现这两者并不一致。

这是因为我们在启动docker的时候，使用了特权模式，或者映射了物理硬件导致的

```
$ sudo docker run 
-- device /dev/card_0
--privileged
```

Docker 本身并不直接负责加载驱动，而是依赖于宿主机（物理机）的系统来完成驱动加载.

所以当映射硬件的时候，是依赖于宿主机的环境的。

当我们更新了宿主机的驱动、依赖库之后，即可正常工作。


## docker 映射硬件的原理

### 宿主机驱动加载

在宿主机上，操作系统会根据硬件设备的存在情况和系统配置来加载相应的驱动程序。这个过程通常在系统启动时完成，或者在检测到新硬件设备插入时动态加载。

Linux 内核使用模块机制来管理驱动程序。驱动程序通常以内核模块（.ko 文件）的形式存在，存放在 /lib/modules/$(uname -r)/kernel/drivers/ 目录下。当系统检测到特定的硬件设备时，会根据设备的识别信息（如 PCI ID、USB ID 等）从 /lib/modules/$(uname -r)/modules.dep 和 /lib/modules/$(uname -r)/modules.alias 等文件中查找对应的内核模块，并通过 modprobe 命令加载该模块。
例如，对于 /dev/biren/card_0 设备，如果它是一个 PCI 设备，系统在启动时会扫描 PCI 总线，识别到该设备的厂商和设备 ID，然后根据这些信息查找并加载对应的内核模块。

udev 是 Linux 下的设备管理器，负责动态创建和管理 /dev 目录下的设备节点。当驱动程序加载后，udev 会根据设备的属性和规则文件（通常位于 /etc/udev/rules.d/ 目录）创建相应的设备节点，如 /dev/biren/card_0。

### Docker 容器与宿主机驱动的关联

Docker 容器通过 --device 选项将宿主机上的设备节点映射到容器内部，使得容器可以直接访问该设备。但容器本身并不负责加载驱动，而是依赖于宿主机已经加载好的驱动。

当使用 --device /dev/biren/card_0 选项时，Docker 会在容器内部创建一个与宿主机上 /dev/biren/card_0 相同的设备节点。这个设备节点只是一个文件，它的作用是作为容器与宿主机上实际硬件设备之间的通信接口。
特权模式与驱动访问
在某些情况下，如果设备需要更高的权限才能访问，可能需要使用 --privileged 选项来启动容器。但即使使用了 --privileged 选项，容器也只是获得了更多的权限来访问宿主机上已经加载好的驱动，而不是在容器内部加载驱动。

### 示例说明

假设宿主机上已经正确安装并加载了 /dev/biren/card_0 设备的驱动程序，并且通过 udev 规则创建了对应的设备节点。当执行以下 Docker 命令时：
bash
sudo docker run --device /dev/biren/card_0 my_image

Docker 会在容器内部创建一个 /dev/biren/card_0 设备节点，该节点与宿主机上的 /dev/biren/card_0 指向同一个硬件设备。
容器内的应用程序可以通过访问 /dev/biren/card_0 设备节点来使用该硬件设备，而驱动程序的加载和管理完全由宿主机负责。
