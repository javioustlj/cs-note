---
title: "tcpdump"
linkTitle: "tcpdump"
weight: 2
draft: false
description: >
    tcpdump usage
params:
  markmap:
    enable: true
---


### 抓取特定协议的二层网络包

```sh
sudo tcpdump -i eth0 -e arp
```

```sh
sudo tcpdump -i eth0 -e ether proto 0x0806
```

协议号在这个链接里面去查找：
https://github.com/torvalds/linux/blob/334426094588f8179fe175a09ecc887ff0c75758/include/uapi/linux/if_ether.h#L132

### 保存到文件

```sh
sudo tcpdump -i eth0 -e arp -w capture.pcap
```
