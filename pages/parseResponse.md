- 代码
  collapsed:: true
	- ```java
	  Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
	      ResponseBody rawBody = rawResponse.body();
	  
	      // Remove the body's source (the only stateful object) so we can pass the response along.
	      rawResponse =
	          rawResponse
	              .newBuilder()
	              .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
	              .build();
	  
	      int code = rawResponse.code();
	      if (code < 200 || code >= 300) {
	        try {
	          // Buffer the entire body to avoid future I/O.
	          ResponseBody bufferedBody = Utils.buffer(rawBody);
	          return Response.error(bufferedBody, rawResponse);
	        } finally {
	          rawBody.close();
	        }
	      }
	  
	      if (code == 204 || code == 205) {
	        rawBody.close();
	        return Response.success(null, rawResponse);
	      }
	  
	      ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
	      try {
	        T body = responseConverter.convert(catchingBody);
	        return Response.success(body, rawResponse);
	      } catch (RuntimeException e) {
	        // If the underlying source threw an exception, propagate that rather than indicating it was
	        // a runtime exception.
	        catchingBody.throwIfCaught();
	        throw e;
	      }
	    }
	  ```
- ## 调用的OkhttpCall的responseConverter.convert转换数据类型，那么需要寻找这个在哪里赋值的
	- 可追溯到 HttpServiceMethod.createResponseConverter 创建ServiceMethod时
	- [[Converter获取流程]]
- ## 则如果添加gson解析，responseConverter则是GsonResponseBodyConverter
	- GsonResponseBodyConverter.convert
		- ```java
		    @Override
		    public T convert(ResponseBody value) throws IOException {
		      JsonReader jsonReader = gson.newJsonReader(value.charStream());
		      try {
		        T result = adapter.read(jsonReader);
		        if (jsonReader.peek() != JsonToken.END_DOCUMENT) {
		          throw new JsonIOException("JSON document was not fully consumed.");
		        }
		        return result;
		      } finally {
		        value.close();
		      }
		    }
		  ```