.. role:: raw-html-m2r(raw)
   :format: html

标签化PLIC
========================

PLIC支持标签化隔离，每个标签对应一组PLIC Gateway和PLIC Target，当前标签的外部中断请求被发送给内核，其他标签标签对应的外部中断被屏蔽并保留。

    - **标签化PLIC Gateway** ： PLIC Gateway只接受外设对应标签产生的中断信号，非本标签的中断不会被采集。
\


    - **标签化PLIC Target** ： Gateway采集中断信号后向对应标签的PLIC Target发送请求，Target对来自本标签的各Gateway的请求根据优先级进行仲裁，得到一个胜出的中断请求。根据当前运行标签对各标签胜出的请求进行选择，只有当前标签对应的请求才会发送给内核，其他标签的请求将会被屏蔽并保留。

    .. code-block:: scala
       :linenos:
       :caption: Gateway

       //每个标签有一组Gateway
        val gateways = for (targetId <- 0 until targetCount) yield
          for(sourceId <- 0 until sourceCount) yield PlicGatewayActiveHigh(
            source = io.sources(targetId)(sourceId),
            id = sourceId + 1,
            priorityWidth = priorityWidth
          )

    .. code-block:: scala
       :linenos:
       :caption: Target

       //每个标签有一个Target
        val targets = for (targetId <- 0 until targetCount) yield PlicTarget(
          id = 0,
          gateways = gateways(targetId),
          priorityWidth = priorityWidth
        )

    .. code-block:: scala
       :linenos:
       :caption: 标签化隔离

       //根据当前标签选择中断信号
        io.interrupt := Vec(targets.map(_.iep))(io.apb.PADDR(32, 3 bits))