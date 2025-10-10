
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
