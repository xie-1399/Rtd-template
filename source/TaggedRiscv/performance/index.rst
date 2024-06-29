===============
性能
===============


.. role:: raw-html-m2r(raw)
   :format: html

使用如下处理器配置及进行性能测试

 - RV64IMACFD指令集。

 - 取指位宽64位，2解码，3发射，2退休流水线结构，发射队列深度为32。

 - 重命名阶段可用物理寄存器个数为64。

 - LSU加载/存储队列深度为16。

 - I/DCache大小32KB，存储与访问位宽为256位，单个Cache行大小256B，2路组相联映射。

 - L2Cache大小256KB，4路组相联映射。

 - 采用BTB、GSHARE、RAS混合分支预测器。

 - 访存延迟为50周期。

 测试性能如下：

 - **Dhrystone**：2.71 DMIPS/Mhz

 - **Coremark**： 4.3 Coremark/Mhz
