- ```java
    static final class CallAdapted<ResponseT, ReturnT> extends HttpServiceMethod<ResponseT, ReturnT> {
      private final CallAdapter<ResponseT, ReturnT> callAdapter;
  
      CallAdapted(
          RequestFactory requestFactory,
          okhttp3.Call.Factory callFactory,
          Converter<ResponseBody, ResponseT> responseConverter,
          CallAdapter<ResponseT, ReturnT> callAdapter) {
        super(requestFactory, callFactory, responseConverter);
        this.callAdapter = callAdapter;
      }
  
      @Override
      protected ReturnT adapt(Call<ResponseT> call, Object[] args) {
        return callAdapter.adapt(call);
      }
    }
  ```
-