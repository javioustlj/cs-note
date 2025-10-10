---
title: "内存序"
linkTitle: "memory order"
weight: 2
draft: false
description: >
    C++ 中的内存序
---

专业术语

- Sequenced-before
- Modification order
- 

## Sequenced-before

1. Sequenced-before是用于描述**同一个线程**中的两个表达式A、B；
2. 如果A is sequenced before B, 那么A 在B开始之前全部完成；
3. 可传递性：如果A is sequenced before B, 且 B is sequenced before C, 那么A is sequenced before C。
