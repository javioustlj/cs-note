---
title: Bash 中常用命令
linkTitle:  Bash 中常用命令
date: 2025-08-27
description: >
     Bash 中常用命令
---

# 文本处理

```sh
for file in succl_debug_ylf-p-sw-br166oam-server1*_*_*.log; do echo "文件: $file"; grep -in -E "Run eagerMode|Async Send|Async Recv" "$file" | wc -l; done
```

```sh
for file in succl_debug_ylf-p-sw-br166oam-server1*_*_*.log; do echo "文件: $file"; grep -in -E "begin func|end func" "$file" | wc -l; done
```

```sh
for file in succl_debug_ylf-p-sw-br166oam-server1*_*_*.log; do echo "文件: $file"; grep -in -E "begin func|end func" "$file" | wc -l; done
```

```sh
for file in succl_debug_ylf-p-sw-br166oam-server1*_*_*.log; do echo "文件: $file"; grep -in -E "Run eagerMode|Async Send|Async Recv" "$file" | wc -l; done
```

```sh
export BRCC_PATH=/path/to/llvm-project
export SUPA_PATH=/path/to/supa
export BESU_PATH=/path/to/BESU
export MPI_HOME=/path/to/openmpi
export LD_LIBRARY_PATH=$SUPA_PATH/lib:$BESU_PATH/lib:/path/to/brml/lib:/path/to/openmpi/lib:$LD_LIBRARY_PATH
```

```sh
xrdb -merge ~/.Xresources
```



# hugo

```sh
hugo server --bind=0.0.0.0 --baseURL=http://10.50.27.103/cs-note/
```


# grep

```sh
grep -A 2 "Pattern 99" SCCL_debug_0_*.log
```

```sh
grep -E "dumpLine.*00/02" -Rn SCCL_debug_0_3193287_0.log
```

匹配多行到一个字符串且是第一个字符串
```sh
^0000[a-f0-9]* <_Z23spcRingReduceScatterOCS.*?(\r\n|\r|\n)(?:(?!end).)*end
```



## linux 系统管理

### 系统版本

```sh
lsb_release -a
```


### sudo 权限管理

```sh
sudoedit /etc/sudoers
```

添加
```
usr1 ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/systemctl
usr ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/systemctl
```
