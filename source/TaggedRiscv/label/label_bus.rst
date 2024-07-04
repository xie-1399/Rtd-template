.. role:: raw-html-m2r(raw)
   :format: html

总线隔离
============================

经过扩展的总线可以携带CPU或DMA发出的标签信息，并通过APB3的鉴权模块与AXI4的二级地址翻译模块实现总线的标签化隔离。

- **APB3总线隔离**
    + **标签位扩展** ： 对PADDR信号的高3位进行高位扩展作为标签位。
    \

    .. image:: /asset/image/label_bus_1.png
    \

    + **地址空间鉴权** ： 访问权限表记录了不同标签对不同地址的访问权限，在使用总线时根据标签位和访问权限表进行鉴权，不合法的访问将返回SlaveError信号，访问权限表由管理程序通过APB3总线配置。

    .. code-block:: scala
       :linenos:
       :caption: 地址空间鉴权

       //访问权限
       val enable = Vec.fill(labelCount)(Vec.fill(mappings.size)(Reg(Bool()) init True))
       //当前标签
       val onehot = mappings.map(_.hit(io.input.PADDR(31 downto 0)))
       //阻止非法访问
       io.output.PENABLE := io.input.PENABLE && enable(io.input.PADDR(34 downto 32))(select)
       //非法访问返回SLVERROR
       io.input.PSLVERROR := io.output.PSLVERROR || !enable(io.input.PADDR(34 downto 32))(select)

- **AXI4总线隔离**
    + **标签位扩展** ： 使用各通道的User信号作为标签位。

    \

    + **二级地址翻译** ： 地址映射表记录了不同标签的可访问地址范围，在使用总线时将原始地址根据地址映射表映射到不同标签的地址空间，并对访问范围进行鉴权，不合法的访问将返回resp信号的DecodeError，地址映射表由管理程序通过APB3总线配置。

    .. code-block:: scala
       :linenos:
       :caption: 二级地址翻译

       //非法访问返回DECERR
       val arexceptionID = Vec(RegInit(False), 1 << idWidth)
       val awexceptionID = Vec(RegInit(False), 1 << idWidth)
       when(io.cachepreTrans.ar.addr(preaddressWidth - 1 downto log2Up(axidataWidth / 8)) > configRegs(U(io.cachepostTrans.ar.user)).regionSize) {
         arexceptionID(io.cachepreTrans.ar.id) := True
       }
       when(io.cachepreTrans.aw.addr(preaddressWidth - 1 downto log2Up(axidataWidth / 8)) > configRegs(U(io.cachepostTrans.aw.user)).regionSize) {
         awexceptionID(io.cachepreTrans.aw.id) := True
         io.cachepostTrans.aw.valid := False
         io.cachepostTrans.w.valid := False
         onWriteID := io.cachepreTrans.aw.id
       }
       when(RegNext(io.cachepreTrans.w.last) && awexceptionID(io.cachepreTrans.aw.id)) {
         io.cachepreTrans.b.valid := True
         io.cachepreTrans.b.id := onWriteID
         io.cachepreTrans.b.resp := resp.DECERR
       }
       when(io.cachepreTrans.b.valid && arexceptionID(io.cachepreTrans.b.id)) {
         io.cachepreTrans.b.resp := resp.DECERR
         arexceptionID(io.cachepreTrans.b.id) := False
       }
