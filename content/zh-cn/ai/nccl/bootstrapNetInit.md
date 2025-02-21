# bootstrapNetInit

作用：找到可用的网络接口，并将它的信息保存下来。

- 如果配置了`NCCL_COMM_ID`，则使用和它在同一网段的网口。
- `ib` 网络接口
- 普通网口
- docker 网口
- loopback 网口


