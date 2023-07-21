- ```kotlin
  fun isHealthy(doExtensiveChecks: Boolean): Boolean {
      assertThreadDoesntHoldLock()
   
      val nowNs = System.nanoTime()
   
      val rawSocket = this.rawSocket!!
      val socket = this.socket!!
      val source = this.source!!
      // 看socket 是否关闭了  
      if (rawSocket.isClosed || socket.isClosed || socket.isInputShutdown ||
              socket.isOutputShutdown) {
        return false
      }
   
      val http2Connection = this.http2Connection
      if (http2Connection != null) {
        // http2是否健康   http2 有ping  pang  验证
        return http2Connection.isHealthy(nowNs)
      }
   
      val idleDurationNs = synchronized(this) { nowNs - idleAtNs }
      if (idleDurationNs >= IDLE_CONNECTION_HEALTHY_NS && doExtensiveChecks) {
        return socket.isHealthy(source)
      }
   
      return true
    }
  ```