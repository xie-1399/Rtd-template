.. role:: raw-html-m2r(raw)
   :format: html

紧耦合存储
============================
TaggedRiscv使用指令紧耦合存储与数据紧耦合存储分离的设计。

- **ITCM：** 用于存储一段固化的程序，ITCM默认情况下为只读。
\

- **DTCM：** DTCM可读可写，支持基于Fence.i指令增强的自定义刷写机制，通过自定义CSR寄存器的值可配置地刷写DTCM。在进行上下文现场保存时，将对应标签的U/S/M CSR与通用寄存器写入DTCM中，恢复时从DTCM中进行读取。