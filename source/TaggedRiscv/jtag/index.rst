===============
调试
===============


.. role:: raw-html-m2r(raw)
   :format: html

TaggedRISCV的外部调试支持JTAG v0.13.2规范，通过OpenOcd进行连接,可通过GDB对处理器上运行的软件进行调试。

调试规范的实现如下：

 - 通过运行一个隐藏的抽象程序缓冲区访问寄存器x0-x31。
\

 - 处理器可以通过CSR读取来读取命令参数，并通过CSR写入提供返回值。
\

 - 触发器（Triggers）只能用作硬件断点

在进行调试功能前，请参考”运行仿真“章节中的内容安装好GDB、OpenOcd等软件。

.. image:: /asset/image/jtag.png
    :align: center
    :scale: 75%