
## 查看一个kernel使用了多少个TLR


在编译选项中的链接选项中加上`-print-reg-count`, 然后在编译的时候就可以看到使用了多少个TLR

```
Function _Z23spcRingReduceScatterOCSILt2ELb0ELb1Eff7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 100 registers
Function _Z23spcRingReduceScatterOCSILt2ELb0ELb1EfDF16b7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 128 registers
Function _Z23spcRingReduceScatterOCSILt2ELb0ELb1Eii7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 100 registers
Function _Z23spcRingReduceScatterOCSILt2ELb0ELb1Ejj7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 100 registers
Function _Z19spcRingAllReduceOCSILt2ELb0ELb1Eff7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 104 registers
Function _Z19spcRingAllReduceOCSILt2ELb0ELb1EfDF16b7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 128 registers
Function _Z19spcRingAllReduceOCSILt2ELb0ELb1Eii7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 104 registers
Function _Z19spcRingAllReduceOCSILt2ELb0ELb1Ejj7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E used 104 registers
```

## 查看一个kernel里面有多少个TLM


```sh
llvm-objdump -d --fatbin libsccl.so
```

可以找到对应的kernel函数的查看其中是否有TLM

```
// cwarp = 2时，Reduce Scatter BF16的TLM
000000000079c260 <_Z23spcRingReduceScatterOCSILt2ELb0ELb1EfDF16b7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E>:
  79c468: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  79c9b8: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  79c9d0: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  79cbe0: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  79cf30: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  79cf50: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  79cf78: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  79d050: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  79d078: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  79d0a8: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r0, g0
  79d0d8: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r4, g0
  79e2e0: 00 a0 82 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e1.gc1 r69, g0
  7a41c0: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r8, g0
  7a4250: 00 a0 82 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e1.gc1 r0, g0
  7a4260: 00 a0 82 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e1.gc1 r1, g0
  7a4278: 00 a0 82 00 ff 0e 16 00      	& ld.tlm.bp1.sint.mb32.rb32.sc2.rw.inter.e1.gc1 r4, g0
  7a4458: 00 a0 86 00 ff 0e 16 00      	& ld.tlm.bp1.sint.mb32.rb32.sc2.rw.inter.e2.gc1 r0, g0
  7a44d0: 00 a0 86 00 ff 0e 16 00      	& ld.tlm.bp1.sint.mb32.rb32.sc2.rw.inter.e2.gc1 r0, g0

// cwarp = 2时，All Reduce BF16的TLM
000000000076bfe0 <_Z19spcRingAllReduceOCSILt2ELb0ELb1EfDF16b7FuncSumEv12ScclWorkElemILN6tensor12MatrixLayoutE0ET2_E>:
  76c148: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76c770: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76c7e8: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76cdb0: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76cdc8: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  76cdf0: 40 a0 81 00 ff 16 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e1.gc2 g0, g1
  76cf20: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76cf48: 40 a0 85 00 ff 1e 06 00      	& st.tlm.bp1.uint.mb32.rb32.rw.inter.e2.gc3 g0, g1
  76cf78: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r4, g0
  76cfa8: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r4, g0
  76e120: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r4, g0
  779660: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r8, g0
  7796f0: 00 a0 86 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e2.gc1 r10, g0
  779700: 00 a0 82 00 ff 0e 16 00      	& ld.tlm.bp1.sint.mb32.rb32.sc2.rw.inter.e1.gc1 r1, g0
  779718: 00 a0 82 00 ff 0e 0e 00      	& ld.tlm.bp1.sint.mb32.rb32.sc1.rw.inter.e1.gc1 r0, g0
  779960: 00 a0 86 00 ff 0e 16 00      	& ld.tlm.bp1.sint.mb32.rb32.sc2.rw.inter.e2.gc1 r0, g0

```


当cwarp = 4时，几个kernel加起来有上千个TLM


想看到代码哪里产生了TLM需要进一步增加brcc的编译选项：`-G -fsupa-device-opt `

反汇编用以下命令：

```sh
llvm-objdump --fatbin -S <a.out>
```

