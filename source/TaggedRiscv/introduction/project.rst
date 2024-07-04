.. role:: raw-html-m2r(raw)
   :format: html

项目目录
============================

本项目的代码构建目录如下：

::

    |-- TaggedRISCV
        |-- .gitignore
        |-- .README.md  // 开发文档
        |-- .ext   // 测试与基准程序
        |-- .src
            |-- main
                |-- scala  // 硬件源码
                    |-- taggedriscv
                        |-- debug  // JTAG接口
                        |-- excute // 执行单元
                        |-- fetch // 取指单元
                        |-- frontend // 流水线前端
                        |-- interface // 硬件模块接口
                        |-- IO // 定制化的IO模块，如DMA、PLIC、CLINT等
                        |-- l2 Cache // 非阻塞L2 Cache
                        |-- labelsAuth // 标签鉴权模块
                        |-- lsu // 访存单元
                        |-- misc // ROB、MMU、CSR、指令提交模块
                        |-- prediction // 分支预测模块
                        |-- riscv // riscv指令格式，寄存器堆
                        |-- sim // 仿真模块
                        |-- twolevelMMU // 二级地址翻译模块
                        |-- utilities // 工具类
                        |-- SOC.scala // 顶层SOC
                        |-- MyConfig.scala // 配置文件
                        |-- Parameters // 参数定义
                |-- tcl    //调试脚本
            |-- test
                |-- scala  //硬件仿真代码
                        |-- SOC // SOC顶层仿真
                        |-- untils // 仿真工具类

        |-- .build.sbt // sbt项目构建
        |-- .test //仿真生成文件，如波形，verilog等

.. image:: /asset/image/parter_new.png
    :align: center
    :scale: 75%