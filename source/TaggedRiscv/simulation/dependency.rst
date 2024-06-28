.. role:: raw-html-m2r(raw)
   :format: html

运行环境及依赖
===============
在项目运行前，需要安装以下依赖：

操作系统
^^^^^^^^^^^^^^
Ubuntu 20.04 LTS

JDK
^^^^^^^^^^^^^^
::

    （1）通过下载安装包并安装：
     http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
     # 转到/opt目录下，此目录用于安装供本系统所有用户使用的软件。
     cd /opt
     # 创建JDK的安装目录
     mkdir JDK1.8
     # 将JDK压缩包移动到安装目录下
     cd JDK1.8
     # 解压JDK压缩包
     mv /home/gao/下载/jdk-8u152-linux-x64.tar.gz ./
     tar -xvf jdk-8u152-linux-x64.tar.gz

     (2) 也可直接通过Ubuntu命令行进行安装
     sudo apt-get update
     sudo apt-get install openjdk-8-jdk

Scala
^^^^^^^^^^^^^^
::

    在Scala官网中下载2.11.12安装包

    sudo tar -zxf scala-2.11.12.tgz -C /usr/local

Sbt
^^^^^^^^^^^^^^

SBT 版本： 1.4.7，安装方式如下：

::

    echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
    echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
    curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
    sudo apt-get update
    sudo apt-get install sbt

如果下载速度较慢时，可以采用华为源
::

    vim ~/.sbt/repositories
    # 粘贴以下内容
    [repositories]
    local
    huaweicloud-maven: https://repo.huaweicloud.com/repository/maven/
    maven-central: https://repo1.maven.org/maven2/
    sbt-plugin-repo: https://repo.scala-sbt.org/scalasbt/sbt-plugin-releases, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]

安装成功后，通过sbt sbtVersion 查看版本号
Verilator or VCS
^^^^^^^^^^^^^^

Verilator版本：4.2.6，安装方式如下:

::

    # Verilator (for sim only, really needs 3.9+, in general apt-get will give you 3.8)
    sudo apt-get install git make autoconf g++ flex bison
    git clone http://git.veripool.org/git/verilator   # Only first time
    unsetenv VERILATOR_ROOT  # For csh; ignore error if on bash
    unset VERILATOR_ROOT  # For bash
    cd verilator
    git pull        # Make sure we're up-to-date
    git checkout v4.216
    autoconf        # Create ./configure script
    ./configure
    sudo make -j8
    sudo make install

同时仿真也可支持VCS 2016 版本

Debug
^^^^^^^^^^^^^^

如果需要通过GDB进行代码调试时，需要安装Openocd + GDB工具：

::

    #Openocd安装
    https://github.com/openocd-org/openocd

    # GDB安装（Prebuild Version）
    version=riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
    wget -O riscv64-unknown-elf-gcc.tar.gz riscv https://static.dev.sifive.com/dev-tools/$version.tar.gz
    tar -xzvf riscv64-unknown-elf-gcc.tar.gz
    sudo mv $version /opt/riscv
    echo 'export PATH=/opt/riscv/bin:$PATH' >> ~/.bashrc

    #GDB安装（交叉工具链编译）
    sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev -y
    git clone --recursive https://github.com/riscv/riscv-gnu-toolchain riscv-gnu-toolchain
    cd riscv-gnu-toolchain
    echo "Starting RISC-V Toolchain build process"
    #compile the  rv64im GCC toolchain
    ARCH=rv64im
    rmdir -rf $ARCH
    mkdir $ARCH; cd $ARCH
    ../configure  --prefix=/opt/$ARCH --with-arch=$ARCH --with-abi=ilp64
    sudo make -j4
    cd ..
    #compile the rv64i GCC toolchain
    ARCH=rv64i
    rmdir -rf $ARCH
    mkdir $ARCH; cd $ARCH
    ../configure  --prefix=/opt/$ARCH --with-arch=$ARCH --with-abi=ilp64
    sudo make -j4
    cd ..
    echo -e "\\nRISC-V Toolchain installation completed!"
