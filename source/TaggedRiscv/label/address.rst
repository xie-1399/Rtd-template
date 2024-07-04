.. role:: raw-html-m2r(raw)
   :format: html

内存隔离
============================

处理器采用位于L2 Cache下的二级地址翻译模块实现多标签下不同程序间物理地址的隔离。

 .. image:: /asset/image/except.png
          :width: 400 px
          :align: center

 - **标签传递**：通过AXI总线中的user（可自定义信号）携带请求所属的标签值。其中标签0属于管理程序，可访问所有的物理空间，并不进行转换。
\
 - **地址二级翻译**：二级地址翻译模块采用多组寄存器保存每个标签所分配内存的起始地址与大小，在请求到来时根据标签值重新计算实际的物理地址并将转换完成的AXI请求发出。

 .. code-block:: scala
   :linenos:
   :caption: 二级地址翻译模块配置寄存器

       case class FliterEntries(val preaddressWidth: Int, val postaddressWidth: Int, val dataWidth: Int) extends Bundle {
           //用于存储各标签地址空间起始地址及范围大小的寄存器
           val beginAddress = Reg(UInt(postaddressWidth bits)) init(0)
           val regionSize = Reg(UInt(preaddressWidth -  log2Up(dataWidth) bits))
       }

\
 - **异常触发**：当访问的地址超过了该标签所分配的内存地址范围，通过AXI总线的RRESP信号向上层Cache返回异常信号，该异常信号存储在Cache的状态位中，待处理器访问该Cache行时，由Fetch单元或LSU单元触发标签越权访问异常进行异常处理。

 .. code-block:: scala
   :linenos:
   :caption: 标签异常触发函数

      //LabelsAuth.scala
      class LabelsAuth extends Plugin with LabelAuthService {
           val completions = ArrayBuffer[Flow[AuthCmd]]()
           override def newLabelExceptPort(codeWidth: Int): Flow[AuthCmd] = completions.addRet(
               //统一标签异常接口，供不同模块调用
               Flow(AuthCmd(
               codeWidth = codeWidth,
               pcWidth = PC_WIDTH
               ))
           )
           val portHits = Bits(completions.size bits)
           val fill = for((port, id) <- completions.zipWithIndex) yield new Area {
               //根据优先级处理
               val others = completions.filter(_.code != port.code)
               portHits(id) := port.valid && !(others.map(o => o.code > port.code && o.valid).orR)
           }
           //异常接口的调用
           val commitPort = setup.commit.newSchedulePort(
               canTrap = true,
               canJump = true,
               causeWidth = 5
           )
           commitPort.valid := False
           when(portHits.orR) {
               commitPort.valid := True
           }
           commitPort.robId := MuxOH.or(portHits, completions.map(_.robId))
           commitPort.trap := True
           commitPort.tval := MuxOH.or(portHits, completions.map(_.tval))
           commitPort.cause := CSR.MCAUSE_ENUM.LABLE_ACCESS_FAULT //自定义异常代码
           commitPort.reason := ScheduleReason.TRAP
           commitPort.skipCommit := True
           commitPort.pcTarget := MuxOH.or(portHits, completions.map(_.targetpc))
       }
     }


\
 - **寄存器配置**：二级地址翻译模块可通过外设总线进行配置，通过总线隔离模块进行访问控制，只允许标签0（管理程序）进行读写以配置每个标签所分配的地址空间及大小。
\
 - **DMA适配**：DMA对内存的访问同样需要经过该模块的处理。
