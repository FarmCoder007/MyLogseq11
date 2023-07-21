- ## newCodec
	- ```kotlin
	    @Throws(SocketException::class)
	    internal fun newCodec(client: OkHttpClient, chain: RealInterceptorChain): ExchangeCodec {
	      val socket = this.socket!!
	      val source = this.source!!
	      val sink = this.sink!!
	      val http2Connection = this.http2Connection
	   
	      return if (http2Connection != null) {
	        // 如果是http2的  我就返回一个 http2的编码解码器  
	        Http2ExchangeCodec(client, this, chain, http2Connection)
	      } else {
	        // 否则返回一个http1的编码解码器
	        socket.soTimeout = chain.readTimeoutMillis()
	        source.timeout().timeout(chain.readTimeoutMillis.toLong(), MILLISECONDS)
	        sink.timeout().timeout(chain.writeTimeoutMillis.toLong(), MILLISECONDS)
	        Http1ExchangeCodec(client, this, source, sink)
	      }
	    }
	  ```
- ### Http1ExchangeCodec
	- ```kotlin
	   /**
	    *  添加请求头
	    */
	    override fun writeRequestHeaders(request: Request) {
	      val requestLine = RequestLine.get(request, connection.route().proxy.type())
	      writeRequest(request.headers, requestLine)
	    }
	   
	     
	    fun writeRequest(headers: Headers, requestLine: String) {
	      check(state == STATE_IDLE) { "state: $state" }
	       // 添加请求行  请求头
	      sink.writeUtf8(requestLine).writeUtf8("\r\n")
	      for (i in 0 until headers.size) {
	        sink.writeUtf8(headers.name(i))
	            .writeUtf8(": ")
	            .writeUtf8(headers.value(i))
	            .writeUtf8("\r\n")
	      }
	      sink.writeUtf8("\r\n")
	      state = STATE_OPEN_REQUEST_BODY
	    }
	  ```