.. role:: raw-html-m2r(raw)
   :format: html

整体架构
==========

- **指令集** : 支持RV64 [I][M][A][C][F][D]
- **流水线** : 采用两路译码、三路执行的乱序超标量流水线结构
- **存储子系统** : 采用二级缓存结构，并配备紧耦合存储
- **外设** ： 支持 Uart、QSPI等外设单元
- **调试** ： 支持 JTAG / OpenOCD / GDB 调试
- **其它** ： 支持RISC-V M/S/U特权模式，SV39虚拟地址管理机制，支持运行Linux操作系统。

TaggedRiscv的整体架构如下图所示：


