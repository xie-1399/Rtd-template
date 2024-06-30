.. role:: raw-html-m2r(raw)
   :format: html

项目目录
============================

本项目的代码构建目录如下：

::

    |-- TaggedRISCV
        |-- .gitignore
        |-- .README.md  //开发文档
        |-- .ext   //测试与基准程序
        |-- .src
            |-- main
                |-- scala  //硬件代码
                |-- tcl    //调试脚本
            |-- test
                |-- scala  //硬件仿真代码
        |-- .build.sbt // sbt项目构建
        |-- .test //仿真生成文件，如波形，verilog等

.. image:: /asset/image/parter_new.png
    :align: center
    :scale: 75%