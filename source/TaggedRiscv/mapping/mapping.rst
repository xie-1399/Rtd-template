.. role:: raw-html-m2r(raw)
   :format: html

现场保存与恢复
============================

处理器采用时分复用的方式执行不同标签程序，在时钟中断到来时，由管理程序进行标签程序的调度，并进行现场的保存与恢复。

 - 通过管理程序将处理器状态保存到数据紧耦合存储（DTCM）中，这些状态包括包括通用寄存器、CSR寄存器等等。
\
 - 在管理程序对通用寄存器进行保存与恢复时，管理程序的执行会破坏当前处理器的状态，如sw t1, 128(t2)指令使用t2寄存器作为基址寄存器，将t1保存到DTCM中时，基址寄存器t2的使用可能会破坏当前标签程序的状态，造成程序错误。
\
 - 针对上述问题，处理器中实例化了两套通用寄存器堆，在寄存器重命名阶段，根据当前处理器的执行状态将逻辑寄存器映射到不同的物理寄存器堆，在执行正常标签程序时，使用通用寄存器堆0进行执行，**在执行管理程序上下文保存程序时，将Load及Store指令的目的寄存器映射到寄存器堆0，将基址寄存器映射到寄存器堆1**，当执行管理程序其他程序时，使用寄存器堆1进行执行。
 .. code-block:: scala
   :linenos:
   :caption: 寄存器堆重映射

   //AllocatorMultiPort.scala
    val ways = for(i <- 0 until waysCount) yield new Area{
    val mem = Mem.fill(entriesPerWay)(dataType) //空闲物理寄存器队列存储结构
    val push = new Area{ //物理寄存器的释放
      //根据CPU状态的不同，采用不同的队列指针
      val offset = (if(isInteger) Mux(io.cpuStatus, ptr.integer.push_spec, ptr.integer.push_normal) else ptr.float.push_normal) + (waysCount-1-i)
      val hits = io.push.map(_.valid).asBits & pushArbitration.push.map(_.ptr(wayRange) === i).asBits
      mem.write(
        //将队列指针进行偏移以选择不同的寄存器堆
        address = if(isInteger) U(io.cpuStatus) @@ offset(addressRange) else offset(addressRange),
        data = MuxOH(hits, io.push.map(_.payload)),
        enable = hits.orR
      )
    }

    val pop = new Area{//物理寄存器的分配
      val offset = (if(isInteger) Mux(io.cpuStatus, ptr.integer.pop_spec, ptr.integer.pop_normal) else ptr.float.pop_normal) + (waysCount-1-i)
      val data = if(isInteger) mem.readAsync(U(io.cpuStatus) @@ offset(addressRange)) else mem.readAsync(offset(addressRange))
      }
    }



 .. image:: /asset/image/phRegistersMapping.png