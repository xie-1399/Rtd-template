.. role:: raw-html-m2r(raw)
   :format: html

MMU
============================
内存管理单元支持SV39规范，采用三级页表，采用二级TLB结构，支持缓存直接命中，支持Refill请求标签化隔离。

    - **支持SV39规范** ： Sv39规范使用39位的虚拟地址，且虚拟地址的高[63:39]位必须和38位保持一致，否则会触发Page Fault。虚拟地址有27位VPN，通过三级页表转换为44位PPN，剩下的12位为offset不变，每一级页号为9位，页表项大小为64位，从而保证一个页表可以被保存在一个页内。
\
    - **二级TLB结构** ： 对于每级页表可以参数化指定地址翻译缓存的直接映射路组的数量。一级TLB采用四路组相联，大小为4KB。二级TLB采用二路组相联，大小为4MB。
\
    - **缓存直接命中** ： 允许ICache直接针对TLB检查其路组tag，可以显著改善时序。
\
    - **标签化** ： MMU发出的Refill请求携带当前处理器正在执行的标签并在Cache中进行标签的比对，来自内存保护单元及外设隔离单元所产生的越权访问交由LSU进行处理，并交由异常处理单元触发非法访问异常。

    .. code-block:: scala
       :linenos:
       :caption: SV39规范

        val sv39 = MmuSpec(
          //三级页表
          levels     = List(
            MmuLevel(virtualWidth = 9, physicalWidth = 9 , virtualOffset =  12, physicalOffset = 12, entryOffset =  10),
            MmuLevel(virtualWidth = 9, physicalWidth = 9 , virtualOffset =  21, physicalOffset = 21, entryOffset =  19),
            MmuLevel(virtualWidth = 9, physicalWidth = 26, virtualOffset =  30, physicalOffset = 30, entryOffset =  28)
          ),
          entryBytes = 8,
          virtualWidth   = 39,
          physicalWidth  = 56,
          satpMode   = 8
        )

    .. code-block:: scala
       :linenos:
       :caption: 二级TLB结构

       val translationStorageParameter = MmuStorageParameter(
        levels   = List(
          MmuStorageLevel(
            id    = 0,
            ways  = 4,
            depth = 32
          ),
          MmuStorageLevel(
            id    = 1,
            ways  = 2,
            depth = 32
          )
        ),
        priority = 1
       )

    .. code-block:: scala
       :linenos:
       :caption: Refill请求标签化

       //Refill CacheLoadCmd
        cmd.valid             := False
        cmd.virtual           := address.resized
        cmd.size              := U(log2Up(spec.entryBytes))
        cmd.redoOnDataHazard  := True
        cmd.unlocked          := False
        cmd.currentlabel      := setup.priv.setup.label //标签