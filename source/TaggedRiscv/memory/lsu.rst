.. role:: raw-html-m2r(raw)
   :format: html

访存单元(LSU)
============================
 - **加载/存储队列（load/Store Queue）**：LSU中使用两个用于Load与Store指令的队列处理来自AGU及写回单元的访存请求，队列深度可配置（默认设置为16）。
\
 - **接收来自AGU的Load指令请求**：执行阶段的AGU发出的Load指令保存在Load队列中并交由LSU内部流水线逻辑发向DCache，当Load队列为空时，AGU发来的请求可直接进入Load流水线中，而无需进入队列，以提高Load指令的执行效率。
\
 - **Store To Load冒险预测器**：对于Store指令，访存的地址与数据均由发射队列提供，后续的Load指令可能会访问当相同的地址，造成Store To Load冒险。为了减少该冒险的产生，引入了Store to Load冒险预测器，根据预测结果，推测性地唤醒Load指令。
\
 - **Store To Load旁路转发**： 当Load流水线中的指令依赖于某条Store指令时，通过数据旁路将数据转发给Load指令，而无需等待Store指令到达写回阶段。
\
 - **并行化地址翻译与数据访问**：对于Load指令，地址翻译与Cache读取并行化进行。
\
 - **标签化**：LSU向Cache发出的Cache请求携带当前处理器正在执行的标签并在Cache中进行标签的比对，LSU对来自内存保护单元及外设隔离单元所产生的越权访问进行处理，交由标签越权管理模块触发非法访问异常。

   .. code-block:: scala
      :linenos:
      :caption: 标签化LSU

      # lsu2Plugin.scala
         val cmd = setup.cacheLoad.cmd
         cmd.currentlabel     := setup.priv.setup.label # 标签通过lsu单元送往Cache
         val currentLabel = setup.priv.setup.label

