.. role:: raw-html-m2r(raw)
   :format: html

智能化DMA
========================

DMA支持AXI4总线上任意模块之间的数据突发传输，设计有双通道同时进行数据传输，自动执行跨页任务分解，主动发起虚拟地址到物理地址的两级翻译，支持标签化隔离，支持任务定时循环触发。

    - **跨页任务分解** ： 执行跨页任务时自动计算地址边界，AXI数据传输会在地址边界停止并于下一页重新发起AXI请求。
\

    - **主动地址翻译** ： 每次发起AXI请求前通过MMU翻译虚拟地址为物理地址，AXI请求通过二级地址翻译模块进行二级地址翻译。
\

    - **标签化隔离** ： 为每个标签配备了一组寄存器，配置DMA时会根据活动标签（APB总线PADDR信号高3位）选择对应寄存器组，DMA根据活动标签执行不同寄存器组的任务，在进行数据搬运时AXI总线的USER信号会携带标签信息。在标签切换时，当前未完成的任务会暂停并将进度会被保存至对应寄存器。
\

    - **定时任务触发** ： 可以配置定时任务，每隔指定时间触发一次数据传输。定时任务有独立于通道任务的寄存器组，定时任务触发时选择空闲通道执行。

    .. image:: /asset/image/dma_arch.png

    .. code-block:: scala
       :linenos:
       :caption: 跨页任务分解

       //根据地址边界自动划分
        val thisLength = Min(
          abortLength - cmdedLength,
          ~virAddress(0, 12 bits) +^ U(1), //距离地址边界的长度
          (burstMlen +^ U(1)) << burstSize
        )

    .. code-block:: scala
       :linenos:
       :caption: 主动地址翻译

       //DMA使用MMU地址翻译接口
        io.trans.cmd.valid := requests.orR
        io.trans.cmd.virtual := virtuals(chosen)
       //MMU地址翻译接口
        val rsp = translation.newTranslationPort(
          stages = stages,
          preAddress = VIRTUAL,
          allowRefill = null,
          usage = LOAD_STORE,
          portSpec = port,
          storageSpec = storage
        )

    .. code-block:: scala
       :linenos:
       :caption: 标签化隔离

       //当前标签
        label := io.apb.PADDR(apb.addressWidth - labelWidth, labelWidth bits)
       //标签独享寄存器组
        val tasks = Seq.fill(channelCount, labelCount)(Task(withTimer = false))
       //标签切换暂停当前任务
        when(valid & normal & label =/= user) {
          read.abortLength := Max(read.cmdedLength, write.cmdedLength)
          write.abortLength := Max(read.cmdedLength, write.cmdedLength)
        }

    .. code-block:: scala
       :linenos:
       :caption: 定时任务触发

       val tasks = Seq.fill(labelCount)(Task(withTimer = true))
       //定时触发逻辑
       val enable = if (withTimer) Reg(Bool()) init False else null
       val time = if (withTimer) Reg(UInt(timerWidth bits)) init U(0) else null
       val counter = if (withTimer) Reg(UInt(timerWidth bits)) init U(0) else null
       if (withTimer) {
         when(enable & !valid) {
           counter := counter + U(1)
         }
         when((counter === time) & enable) {
           valid := True
         }
         when(valid || !enable) {
           counter := U(0)
         }
         when(time =/= RegNext(time)) {
           enable.clear()
         }
       }