.. role:: raw-html-m2r(raw)
   :format: html

环境变量
================

JDK环境变量
^^^^^^^^^^^^^^
::

    # 打开环境变量配置文件/etc/profile,在此文件中修改的配置将对本系统所有用户有效。
    gedit /etc/profile
    # 添加JDK配置信息
    export JAVA_HOME=/opt/JDK1.8/jdk1.8.0_152
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH
    # 使用source命令重新读取并执行profile中的配置信息
    source /etc/profile
    # 测试,执行如下命令查看JDK版本，如果返回当前JDK版本信息则说明安装成功
    java -version



Scala环境变量
^^^^^^^^^^^^^^
::

    export SCALA_HOME=/usr/local/scala-2.11.12
    export PATH=$SCALA_HOME/bin:$PATH
    Source ~/.bashrc
    scala -version
