.. role:: raw-html-m2r(raw)
   :format: html

缓存隔离
============================

在一级及二级缓存中，通过添加标签的形式，实现缓存的多标签隔离，提高数据安全性。

 - **空间上**：在缓存状态中添加标签位，在进行Tag比对时，额外对比Label是否匹配，不同标签同一位置的行视为不一样的行，在标签未命中时，同样触发缓存缺失。
\
 .. code-block:: scala
   :linenos:
   :caption: 自定义Fence.i指令冲刷

   //EnvCallPlugin.scala
      val flushes = new StateMachine{
      val flushSignal = setup.priv.setup.flushSignal
      val currentlabel = setup.priv.setup.label
      val vmaPort = getService[AddressTranslationService].invalidatePort
      val fetchPort = getService[FetchCachePlugin].invalidatePort
      val lsuPort   = getServiceOption[LsuFlusher] match {
        case Some(lsu) => lsu.getFlushPort()
        case None => println("No LSU plugin for the EnvCallPlugin flush ???"); null
      }

      val IDLE, RESCHEDULE, VMA_FETCH_FLUSH, VMA_FETCH_WAIT = new State()
      val LSU_FLUSH, WAIT_LSU, WAIT_L2CACHE_TCM  = (lsuPort != null) generate new State
      setEntry(IDLE)

      fetch.getStage(0).haltIt(!isActive(IDLE))

      val vmaInv, fetchInv, flushData = Reg(Bool())

      IDLE whenIsActive{
        when(isValid) {
          vmaInv := FENCE_VMA
          fetchInv := FENCE_I
          flushData := FLUSH_DATA
          when(isReady) {
            when(FENCE_I || FENCE_VMA) {
                goto(RESCHEDULE)
            }
            when(FLUSH_DATA) {  //进入数据冲刷阶段
              goto(if(lsuPort != null) LSU_FLUSH else IDLE)
            }
          }
        }
      }

      RESCHEDULE whenIsActive{
        when(commit.reschedulingPort(onCommit = true).valid){
          goto(VMA_FETCH_FLUSH)
        }
      }

      VMA_FETCH_FLUSH whenIsActive {
        when(flushSignal(5)) {
          when(commit.isRobEmpty) {  //排空流水线
            goto(IDLE)
          }
        }.otherwise {
          vmaPort.cmd.valid setWhen (vmaInv)
          fetchPort.cmd.valid setWhen (fetchInv)
          fetchPort.cmd.payload := flushSignal(0).asUInt
          goto(VMA_FETCH_WAIT)
        }
      }

      VMA_FETCH_WAIT whenIsActive{
        when(fetchPort.rsp.valid) {
          goto(if(lsuPort != null) LSU_FLUSH else IDLE)
        }
        when(vmaPort.rsp.valid){
          goto(IDLE)
        }
      }

      if(lsuPort != null) {
        LSU_FLUSH whenIsActive {
          lsuPort.cmd.valid := True
          lsuPort.cmd.withFree := flushData || flushSignal(3) //冲刷L1 Cache
          goto(WAIT_LSU)
        }

        WAIT_LSU whenIsActive {
          when(lsuPort.rsp.valid) {
            when((!flushSignal(4)) && (!flushSignal(7))) { // 根据CSR冲刷L2 Cache或者DTCM
              goto(IDLE)
            } otherwise {
              when(flushSignal(7) && flushSignal(4)){
                l2CacheFlush := True
                l2CacheFlushLabels := B(currentlabel)
                tcmFlush := True
                goto(WAIT_L2CACHE_TCM)
              }.otherwise{
                when(flushSignal(4)) {
                  l2CacheFlush := True
                  l2CacheFlushLabels := B(currentlabel)
                  goto(WAIT_L2CACHE_TCM)
                }
                when(flushSignal(7)) {
                  tcmFlush := True
                  goto(WAIT_L2CACHE_TCM)
                }
              }
            }
          }
        }

        WAIT_L2CACHE_TCM whenIsActive {  //等待DTCM或L2 Cache冲刷完成
          when(flushSignal(7) && flushSignal(4)){
            when(l2CacheFlushReady && tcmFlushReady){
              goto(IDLE)
            }
          }.elsewhen(flushSignal(7)){
            when(tcmFlushReady){
              goto(IDLE)
            }
          }.otherwise{
            when(l2CacheFlushReady) {
              goto(IDLE)
            }
          }
         }
      }

      build()
    }


 - **时间上**：增强Fence.i指令，在进行标签程序切换时，可通过自定义CSR寄存器（0xBC4）将该属于该标签的缓存行设为无效。

 .. image:: /asset/image/cacheIso.png
         :width: 500 px
         :align: center





