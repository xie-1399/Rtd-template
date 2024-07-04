.. role:: raw-html-m2r(raw)
   :format: html

标签化TLB
============================

TaggedRISCV支持SV39规范，采用了二级TLB结构。

    - **一级TLB：** 一级TLB采用四路组相联，大小为4 KB。
\

    - **二级TLB：** 二级TLB采用二路组相联，大小为4 MB。

TLB表项如下图所示：

    .. image:: /asset/image/TLB_new.png
      :align: center


在TLB表项中增加标签位表项，在不同标签的虚拟地址进行转化时，通过对比标签位，从而实现不同标签下虚拟地址的隔离。

.. code-block:: scala
   :linenos:
   :caption: 标签化TLB

   // MMU.scala
   cmd.currentlabel      := setup.priv.setup.label # 获取当前标签
   when(requireMmuLockup) {
          REDO          := !hit
          TRANSLATED    := lineTranslated
          ID            := setup.priv.setup.label
          ALLOW_EXECUTE := lineAllowExecute && !(lineAllowUser && priv.isSupervisor())
          ALLOW_READ    := lineAllowRead || status.mxr && lineAllowExecute
          ALLOW_WRITE   := lineAllowWrite
          PAGE_FAULT    := lineException || lineAllowUser && priv.isSupervisor() && !status.sum || !lineAllowUser && priv.isUser()
          ACCESS_FAULT  := lineAccessFault
        } otherwise {
          REDO          := False
          ID            := setup.priv.setup.label
          TRANSLATED    := ps.preAddress.resized
          ALLOW_EXECUTE := True
          ALLOW_READ    := True
          ALLOW_WRITE   := True
          PAGE_FAULT    := False
          ACCESS_FAULT  := ps.preAddress.drop(physicalWidth) =/= 0
        }