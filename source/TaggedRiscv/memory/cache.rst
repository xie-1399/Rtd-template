.. role:: raw-html-m2r(raw)
   :format: html

两级Cache
============================

处理器Cache采用两级Cache结构，均为私有Cache，两级Cache间通过AXI总线协议进行数据交换。

I Cache
--------------------------
 ICache与Fetch阶段紧耦合，根据PC值读取指令。

 - **配置**：ICache大小及映射路数可配置，默认采用2路组相联映射方式，大小为32KB,行大小256B，随机替换策略。
\
 - **标签化**：ICache进行了标签化处理，在状态位中存储该Cache行所属的标签，当读取Tag并进行比对的同时，读取Label并与当前处理器所执行的标签进行比对，当Tag命中但Label命中时，视为未命中，触发Cache Miss，进行Refill。
\
 - **自定义刷写**：ICache支持基于Fence.i指令增强的自定义刷写机制，通过自定义CSR寄存器（0xBC4）的值可配置地刷写ICache。

D Cache
--------------------------
 DCache接收来自LSU的请求，处理加载及存储指令，通过两条流水线分别处理加载与存储指令。
\
 - **配置**：DCache的大小及映射路数可配置，默认采用2路组相联映射方式，大小为32KB，行大小256B，随机替换策略。
\
 - **非阻塞**：DCache为非阻塞结构，采用Refiil Slots及WriteBack Slots记录并处理未命中的请求，其深度都为4，DCache未命中的请求会在发起Refill请求之后，向LSU返回REDO信号，并在装载数据完成后，向LSU返回完成信号。
\
 - **标签化**：DCache同样进行了标签化处理，在状态位中存储该Cache行所属的标签，当读取Tag并进行比对的同时，读取Label并与当前处理器所执行的标签进行比对，当Tag命中但Label命中时，视为未命中，触发Cache Miss，进行Refill。
\
 - **自定义刷写**：DCache支持基于Fence.i指令增强的自定义刷写机制，通过自定义CSR寄存器（0xBC4）的值可配置地刷写DCache。

L2 Cache
--------------------------
 L2Cache接收并通过两条流水线分别处理来自I/D Cache的请求，请求采用标准AXI总线协议进行传输。

 - **非阻塞**：L2Cache为非阻塞结构，支持上层请求的乱序返回，采用Refiil Slots及WriteBack Slots存储未命中的请求，其深度都为8。
\
 - **并行访问**：数据、标记、状态位均采用基于Dual-Port RAM的多Bank存储结构，采用Request Queue统一调度多个数据访问请求，支持多Bank多端口并行读写。
\
 - **一致性**：两级Cache之间采用Inclusive包含策略，在L2Cache替换时询问上层Cache状态并进行写回与无效处理以维持一致性。
\
 - **DMA适配**：SOC中的DMA直接内存访问会带来内存与Cache间的一致性问题，支持来自DMA的行无效请求，将一页地址范围的Cache行设为无效，并通过一致性策略维护上下层Cache间的一致性。
\
 - **标签化处理**：L2Cache同样进行了标签化处理，在状态位中存储该Cache行所属的标签，当读取Tag并进行比对的同时，读取Label并与当前处理器所执行的标签进行比对，当Tag命中但Label命中时，视为未命中，触发Cache Miss，进行Refill。
\
 - **自定义刷写**：L2Cache同样支持自定义的数据刷写。

.. image:: /asset/image/L2Cache.png
        :scale: 85%
        :align: center




