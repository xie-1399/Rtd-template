.. role:: raw-html-m2r(raw)
   :format: html

在IDEA中运行仿真
============================

在IDEA上可按照下面的流程进行配置

    （1）IDEA官网https://www.jetbrains.com/idea/下载安装包。

    （2）利用idea打开项目后，在File -> Project Structure中配置jdk的路径。

    .. image:: /asset/image/simulation1.png
       :align: center

    （3）setting -> plugin中的market搜索Scala插件，并点击安装，安装完成后重启idea。

    .. image:: /asset/image/simulation2.png
       :align: center

    （4）build.sbt文件中，点击build后，将会自动下载项目对应的依赖。

    （5）运行测试文件，运行测试后生成的波形文件与verilog文件位于根目录的test文件夹下。

    .. image:: /asset/image/simulation3.png
      :align: center