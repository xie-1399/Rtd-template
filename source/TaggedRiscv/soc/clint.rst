.. role:: raw-html-m2r(raw)
   :format: html

标签化CLINT
========================

    - **标签化隔离** ： 每个标签对应一组定时中断和软件中断，活动标签（APB总线PADDR高3位）对应的定时中断和软件中断被发送给内核，非活动标签对应的定时中断和软件中断被屏蔽并保留。
\

    - **标签时间片中断** ： 用于给标签时间片定时的中断，标签时间片结束时发出定时中断，时间片长度由管理程序配置，其他标签可以通过写入寄存器提前结束时间片。

    .. code-block:: scala
       :linenos:
       :caption: 标签化隔离

        val harts = for (index <- 0 until hartCount) yield new Area {
          val cmp = Reg(UInt(64 bits)) init U(0x7FFFFFFF)
          val timerInterrupt = RegNext(time >= cmp)
          val softwareInterrupt = RegInit(False)
        }

    .. code-block:: scala
       :linenos:
       :caption: 标签时间片中断

        val switch = new Area {
          val label = io.apb.PADDR(34 downto 32)
          val cmp = Reg(UInt(64 bits)) init U(0x7FFFFFFF)
          val avant = RegInit(False)
          val timerInterrupt = RegNext(time >= cmp || avant)
        }