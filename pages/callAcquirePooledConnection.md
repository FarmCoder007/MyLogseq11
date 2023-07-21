- ```java
  
  // 尝试获取到呼叫地址的回收连接。如果已获取连接，则返回true。  
  // 参数1:ip地址
  // 参数2：realCall对象
  // 参数3：路由列表route  路由里包含  ip地址InetSocketAddress  
  //     代理模式Proxy        端口Address这里包含端口 
  // 参数4：是否需要多路复用 
  // 是否让多个请求 共享一个连接  ，，是否只拿多路复用的连接
  fun callAcquirePooledConnection(
      address: Address,
      call: RealCall,
      routes: List<Route>?,
      requireMultiplexed: Boolean
    ): Boolean {
      for (connection in connections) {
        synchronized(connection) {
          //注释2： 判断连接的多路复用，这个属性是给Http2用的
          if (requireMultiplexed && !connection.isMultiplexed) return@synchronized
          // 检查ip地址和路由列表是否合法
          if (!connection.isEligible(address, routes)) return@synchronized
          // 调用 RealCall.acquireConnectionNoEvents()方法，将RealCall的 connection指向该连接，
       //表明存在可以复用的连接，并且返回true。那么调用者就可以通过它的RealCall来获取到复用的连接了
          call.acquireConnectionNoEvents(connection)
          return true
        }
      }
      return false
    }
  ```
- ## 参数讲解
	- address：
		- 主机名 和 端口
	- routes
		- 具体的路由 比如
		- ip地址 ：InetSocketAddress     代理模式Proxy        端口Address这里包含端口
		- ## 用处，下边isEligible 判断合规，http2，链接合并用
	- requireMultiplexed
		- 是否需要多路复用
- RealConnection.isEligible 判断是否是匹配的
	- ```java
	  internal fun isEligible(address: Address, routes: List<Route>?): Boolean {
	      assertThreadHoldsLock()
	  
	      // 1、这个链接所承受的call 请求数量 没有超限制 默认值allocationLimit = 1   http2 这个限制会大些
	      // 2、noNewExchanges 这个链接还可用，还可以接受新的请求
	      if (calls.size >= allocationLimit || noNewExchanges) return false
	  
	    
	        //  地址不同，不能复用，包括配置的dns 代理 证书 端口等
	      if (!this.route.address.equalsNonHost(address)) return false
	  
	      // 如果主机 完全匹配就可以复用 
	      if (address.url.host == this.route().address.url.host) {
	        return true // This connection is a perfect match.
	      }
	  
	      // At this point we don't have a hostname match. But we still be able to carry the request if
	      // our connection coalescing requirements are met. See also:
	      // https://hpbn.co/optimizing-application-delivery/#eliminate-domain-sharding
	      // https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/
	      // 从这开始 http2的
	      // 1. This connection must be HTTP/2.
	      if (http2Connection == null) return false
	  
	      // 2. The routes must share an IP address.
	      if (routes == null || !routeMatchesAny(routes)) return false
	  
	      // 3. This connection's server certificate's must cover the new host.
	      if (address.hostnameVerifier !== OkHostnameVerifier) return false
	      if (!supportsUrl(address.url)) return false
	  
	      // 4. Certificate pinning must match the host.
	      try {
	        address.certificatePinner!!.check(address.url.host, handshake()!!.peerCertificates)
	      } catch (_: SSLPeerUnverifiedException) {
	        return false
	      }
	  
	      return true // The caller's address can be carried by this connection.
	    }
	  ```
	-
-