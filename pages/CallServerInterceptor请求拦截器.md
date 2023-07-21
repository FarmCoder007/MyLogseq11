- CallServerInterceptor ，利用 HttpCodec 发出请求到服务器并且解析生成 Response 。
- 首先调用 httpCodec.writeRequestHeaders(request); 将请求头写入到缓存中(直到调用 flushRequest() 才真正发送给服务器)。然后马上进行第一个逻辑判断
  collapsed:: true
	- ```java
	     Response.Builder responseBuilder = null;
	      if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
	  // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
	  // Continue" response before transmitting the request body. If we don't get that, return
	  // what we did get (such as a 4xx response) without ever transmitting the request body.
	          if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
	              httpCodec.flushRequest();
	              realChain.eventListener().responseHeadersStart(realChain.call());
	              responseBuilder = httpCodec.readResponseHeaders(true);
	          }
	          if (responseBuilder == null) {
	  // Write the request body if the "Expect: 100-continue" expectation was met.
	              realChain.eventListener().requestBodyStart(realChain.call());
	              long contentLength = request.body().contentLength();
	              CountingSink requestBodyOut =
	                      new CountingSink(httpCodec.createRequestBody(request, contentLength));
	              BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
	              request.body().writeTo(bufferedRequestBody);
	              bufferedRequestBody.close();
	              realChain.eventListener().requestBodyEnd(realChain.call(),requestBodyOut.successfulCount);
	          } else if (!connection.isMultiplexed()) {
	  //HTTP2多路复用，不需要关闭socket，不管！
	  // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1
	  // connection
	  // from being reused. Otherwise we're still obligated to transmit the request
	  // body to
	  // leave the connection in a consistent state.
	              streamAllocation.noNewStreams();
	          }
	      }
	      httpCodec.finishRequest();
	  ```
- 整个if都和一个请求头有关： Expect: 100-continue 。这个请求头代表了在发送请求体之前需要和服务器确定是否愿意接受客户端发送的请求体。所以 permitsRequestBody 判断为是否会携带请求体的方式(POST)，如果命中if，则会先给服务器发起一次查询是否愿意接收请求体，这时候如果服务器愿意会响应100(没有响应体，responseBuilder 即为nul)。这时候才能够继续发送剩余请求数据。
- 但是如果服务器不同意接受请求体，那么我们就需要标记该连接不能再被复用，调用 noNewStreams() 关闭相关的Socket。
- 后续代码为:
	- ```java
	   if (responseBuilder == null) {
	          realChain.eventListener().responseHeadersStart(realChain.call());
	          responseBuilder = httpCodec.readResponseHeaders(false);
	      }
	      Response response = responseBuilder
	              .request(request)
	              .handshake(streamAllocation.connection().handshake())
	              .sentRequestAtMillis(sentRequestMillis)
	              .receivedResponseAtMillis(System.currentTimeMillis())
	              .build();
	  ```
- 这时 responseBuilder 的情况即为：
	- 1、POST方式请求，请求头中包含 Expect ，服务器允许接受请求体，并且已经发出了请求体， responseBuilder为null;
	- 2、POST方式请求，请求头中包含 Expect ，服务器不允许接受请求体， responseBuilder 不为null
	- 3、POST方式请求，未包含 Expect ，直接发出请求体， responseBuilder 为null;
	- 4、POST方式请求，没有请求体， responseBuilder 为null;
	- 5、GET方式请求， responseBuilder 为null
- 对应上面的5种情况，读取响应头并且组成响应 Response ，注意：此 Response 没有响应体。同时需要注意的是，如果服务器接受 Expect: 100-continue 这是不是意味着我们发起了两次 Request ？那此时的响应头是第一次查询服务器是否支持接受请求体的，而不是真正的请求对应的结果响应。所以紧接着
	- ```java
	   int code = response.code();
	      if (code == 100) {
	  // server sent a 100-continue even though we did not request one.
	  // try again to read the actual response
	          responseBuilder = httpCodec.readResponseHeaders(false);
	          response = responseBuilder
	                  .request(request)
	                  .handshake(streamAllocation.connection().handshake())
	                  .sentRequestAtMillis(sentRequestMillis)
	                  .receivedResponseAtMillis(System.currentTimeMillis())
	                  .build();
	          code = response.code();
	      }
	  ```
- 如果响应是100，这代表了是请求 Expect: 100-continue 成功的响应，需要马上再次读取一份响应头，这才是真正的请求对应结果响应头。
- 然后收尾
  collapsed:: true
	- ```java
	    if (forWebSocket && code == 101) {
	  // Connection is upgrading, but we need to ensure interceptors see a non-null
	  // response body.
	          response = response.newBuilder()
	                  .body(Util.EMPTY_RESPONSE)
	                  .build();
	      } else {
	          response = response.newBuilder()
	                  .body(httpCodec.openResponseBody(response))
	                  .build();
	      }
	      if ("close".equalsIgnoreCase(response.request().header("Connection"))
	              || "close".equalsIgnoreCase(response.header("Connection"))) {
	          streamAllocation.noNewStreams();
	      }
	      if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
	          throw new ProtocolException(
	                  "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
	      }
	      return response；
	  ```
- forWebSocket 代表websocket的请求，我们直接进入else，这里就是读取响应体数据。然后判断请求和服务器是不是都希望长连接，一旦有一方指明 close ，那么就需要关闭 socket 。而如果服务器返回204/205，一般情况而言不会存在这些返回码，但是一旦出现这意味着没有响应体，但是解析到的响应头中包含 Content-Lenght 且不为0，这表响应体的数据字节长度。此时出现了冲突，直接抛出协议异常！
- ## **总结**
	- 在这个拦截器中就是完成HTTP协议报文的封装与解析。
- ## [[请求拦截器面试题]]