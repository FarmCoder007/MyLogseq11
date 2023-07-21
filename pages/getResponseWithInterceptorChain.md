- ## 代码
	- ```java
	   @Throws(IOException::class)
	    internal fun getResponseWithInterceptorChain(): Response {
	      // Build a full stack of interceptors.
	      val interceptors = mutableListOf<Interceptor>()
	      // 1、先添加自定义拦截器  
	      interceptors += client.interceptors
	      // 2、五大内置拦截器  
	      interceptors += RetryAndFollowUpInterceptor(client)
	      interceptors += BridgeInterceptor(client.cookieJar)
	      interceptors += CacheInterceptor(client.cache)
	      interceptors += ConnectInterceptor
	      if (!forWebSocket) {
	        interceptors += client.networkInterceptors
	      }
	      interceptors += CallServerInterceptor(forWebSocket)
	  
	      val chain = RealInterceptorChain(
	          call = this,
	          interceptors = interceptors,
	          index = 0,
	          exchange = null,
	          request = originalRequest,
	          connectTimeoutMillis = client.connectTimeoutMillis,
	          readTimeoutMillis = client.readTimeoutMillis,
	          writeTimeoutMillis = client.writeTimeoutMillis
	      )
	  
	      var calledNoMoreExchanges = false
	      try {
	        // 3、责任链 执行 各个拦截器 返回 响应
	        val response = chain.proceed(originalRequest)
	        if (isCanceled()) {
	          response.closeQuietly()
	          throw IOException("Canceled")
	        }
	        return response
	      } catch (e: IOException) {
	        calledNoMoreExchanges = true
	        throw noMoreExchanges(e) as Throwable
	      } finally {
	        if (!calledNoMoreExchanges) {
	          noMoreExchanges(null)
	        }
	      }
	    }
	  ```
- ## 1、自定义拦截器 [[#red]]==interceptors== 加到最前边
- ## 2、4个内置拦截器
- ## 3、自定义的webSocket，[[#red]]==networkInterceptors==拦截器
- ## 4、最后一个[[#red]]==CallServerInterceptor==内置拦截器