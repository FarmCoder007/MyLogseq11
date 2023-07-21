- ## 代码
	- ```java
	  @Throws(IOException::class)
	    private fun findConnection(
	      connectTimeout: Int,
	      readTimeout: Int,
	      writeTimeout: Int,
	      pingIntervalMillis: Int,
	      connectionRetryEnabled: Boolean
	    ): RealConnection {
	      if (call.isCanceled()) throw IOException("Canceled")
	   
	      // Attempt to reuse the connection from the call.
	      val callConnection = call.connection // This may be mutated by releaseConnectionNoEvents()!
	  	// 当call里有链接的时候  看这个链接  ip啊 端口啊路径啊 和自己将要请求的是否一样 
	      // 不一样将链接 释放掉
	      if (callConnection != null) {
	        var toClose: Socket? = null
	        synchronized(callConnection) {
	          if (callConnection.noNewExchanges || !sameHostAndPort(callConnection.route().address.url)) {
	            toClose = call.releaseConnectionNoEvents()
	          }
	        }
	   
	        // If the call's connection wasn't released, reuse it. We don't call connectionAcquired() here
	        // because we already acquired it.
	        // 第五次  call里有链接  经过上边的检验  call里的connection还不为空的话 就可以复用了  
	        if (call.connection != null) {
	          check(toClose == null)
	          return callConnection
	        }
	   
	        // The call's connection was released.
	        toClose?.closeQuietly()
	        eventListener.connectionReleased(call, callConnection)
	      }
	   
	      // We need a new connection. Give it fresh stats.
	      refusedStreamCount = 0
	      connectionShutdownCount = 0
	      otherFailureCount = 0
	   
	      // Attempt to get a connection from the pool.
	      // 第一次尝试拿连接   拿事先放在 池里的连接
	      if (connectionPool.callAcquirePooledConnection(address, call, null, false)) {
	        val result = call.connection!!
	        eventListener.connectionAcquired(call, result)
	        return result
	      }
	   
	      // Nothing in the pool. Figure out what route we'll try next.
	      val routes: List<Route>?
	      val route: Route
	      if (nextRouteToTry != null) {
	        // Use a route from a preceding coalesced connection.
	        routes = null
	        route = nextRouteToTry!!
	        nextRouteToTry = null
	      } else if (routeSelection != null && routeSelection!!.hasNext()) {
	        // Use a route from an existing route selection.
	        routes = null
	        route = routeSelection!!.next()
	      } else {
	        // Compute a new route selection. This is a blocking operation!
	        var localRouteSelector = routeSelector
	        if (localRouteSelector == null) {
	          localRouteSelector = RouteSelector(address, call.client.routeDatabase, call, eventListener)
	          this.routeSelector = localRouteSelector
	        }
	        val localRouteSelection = localRouteSelector.next()
	        routeSelection = localRouteSelection
	        routes = localRouteSelection.routes
	   
	        if (call.isCanceled()) throw IOException("Canceled")
	   
	        // Now that we have a set of IP addresses, make another attempt at getting a connection from
	        // the pool. We have a better chance of matching thanks to connection coalescing.
	        // 第二次尝试  从池里拿  但是这次传入个参数 routes
	        if (connectionPool.callAcquirePooledConnection(address, call, routes, false)) {
	          val result = call.connection!!
	          eventListener.connectionAcquired(call, result)
	          return result
	        }
	   
	        route = localRouteSelection.next()
	      }
	   
	      // Connect. Tell the call about the connecting call so async cancels work.
	      // 第三次  前两次都拿不到链接   自己创建链接
	      val newConnection = RealConnection(connectionPool, route)
	      call.connectionToCancel = newConnection
	      try {
	        //  这里是自己创建的  RealConnection  做链接
	        newConnection.connect(
	            connectTimeout,
	            readTimeout,
	            writeTimeout,
	            pingIntervalMillis,
	            connectionRetryEnabled,
	            call,
	            eventListener
	        )
	      } finally {
	        call.connectionToCancel = null
	      }
	      call.client.routeDatabase.connected(newConnection.route())
	   
	      // If we raced another call connecting to this host, coalesce the connections. This makes for 3
	      // different lookups in the connection pool!
	       // 第四次 准备把新建的 放在连接池里 前  再尝试  从池里获取一次链接
	      if (connectionPool.callAcquirePooledConnection(address, call, routes, true)) {
	        val result = call.connection!!
	        nextRouteToTry = route
	        newConnection.socket().closeQuietly()
	        eventListener.connectionAcquired(call, result)
	        return result
	      }
	   
	      synchronized(newConnection) {
	        connectionPool.put(newConnection)
	        call.acquireConnectionNoEvents(newConnection)
	      }
	   
	      eventListener.connectionAcquired(call, newConnection)
	      return newConnection
	    }
	  ```
- ## [[第四次获取自己创建链接的流程]]
- ## 总结，获取一个可用链接的流程
	- ## [[链接池复用的流程]]
	-