- ## 1、HttpServiceMethod.createResponseConverter
	- ```java
	    private static <ResponseT> Converter<ResponseBody, ResponseT> createResponseConverter(
	        Retrofit retrofit, Method method, Type responseType) {
	      Annotation[] annotations = method.getAnnotations();
	      try {
	        return retrofit.responseBodyConverter(responseType, annotations);
	      } catch (RuntimeException e) { // Wide exception range because factories are user code.
	        throw methodError(method, e, "Unable to create converter for %s", responseType);
	      }
	    }
	  ```
- ## 2、retrofit.responseBodyConverter
	- ```java
	   public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
	      return nextResponseBodyConverter(null, type, annotations);
	    }
	  
	  public <T> Converter<ResponseBody, T> nextResponseBodyConverter(
	        @Nullable Converter.Factory skipPast, Type type, Annotation[] annotations) {
	      Objects.requireNonNull(type, "type == null");
	      Objects.requireNonNull(annotations, "annotations == null");
	  
	      int start = converterFactories.indexOf(skipPast) + 1;
	      for (int i = start, count = converterFactories.size(); i < count; i++) {
	        Converter<ResponseBody, ?> converter =
	            converterFactories.get(i).responseBodyConverter(type, annotations, this);
	        if (converter != null) {
	          //noinspection unchecked
	          return (Converter<ResponseBody, T>) converter;
	        }
	      }
	  
	      StringBuilder builder =
	          new StringBuilder("Could not locate ResponseBody converter for ")
	              .append(type)
	              .append(".\n");
	      if (skipPast != null) {
	        builder.append("  Skipped:");
	        for (int i = 0; i < start; i++) {
	          builder.append("\n   * ").append(converterFactories.get(i).getClass().getName());
	        }
	        builder.append('\n');
	      }
	      builder.append("  Tried:");
	      for (int i = start, count = converterFactories.size(); i < count; i++) {
	        builder.append("\n   * ").append(converterFactories.get(i).getClass().getName());
	      }
	      throw new IllegalArgumentException(builder.toString());
	    }
	  ```
- ## GsonConverterFactory的.responseBodyConverter
	- ```java
	    public Converter<ResponseBody, ?> responseBodyConverter(
	        Type type, Annotation[] annotations, Retrofit retrofit) {
	      TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
	      return new GsonResponseBodyConverter<>(gson, adapter);
	    }
	  ```
- ## 总结
	- 同样的配方，从Retrofit build 时，初始化设置的converterFactories 里 根据 响应type去取 相应的converter，应该是Retrofit 初始化时 手动添加的GsonConverterFactory,返回的[[#red]]==**GsonResponseBodyConverter**==
	-