.. role:: raw-html-m2r(raw)
   :format: html

标签化流水线
============================

使用一个自定义CSR寄存器存储当前流水线中的标签，CSR地址为0xBC0，CSR结构如下图所示，最多可存储8个标签结构，其中标签0分配给管理程序。

.. image:: /asset/image/label_csr.png
    :align: center
    :scale: 80%

- **标签读写权限：** 标签CSR寄存器是可读的，但对于标签CSR寄存器的写需要进行权限判断，对该CSR进行写操作时会判断标签是否为管理程序的标签，如果是管理程序的标签才允许写，否则不允许。

- **访存单元：** 访存单元是流水线中标签传递到下级存储设备的单元，在执行单元（AGU）计算出访存地址后，访存请求会携带标签的值（即自定义标签CSR的值），在Cache、TLB、总线中进行传递。

.. image:: /asset/image/label_lsu.png
    :align: center
    :scale: 80%
