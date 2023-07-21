- ```java
  //step1
  Retrofit retrofit = new Retrofit.Builder()
                      .baseUrl("https://www.wanandroid.com/")
                      .addConverterFactory(GsonConverterFactory.create(new Gson()))
                      .build();
  //step2
  ISharedListService sharedListService = retrofit.create(ISharedListService.class);
  //step3
  Call<SharedListBean> sharedListCall = sharedListService.getSharedList(2,1);
  ```
- ### 1、retrofit.create采用动态代理在内存中创建服务接口的代理类，反射创建代理类对象，
- ### 2、代理类构造函数中会传入InvocationHandler的回调接口。同时实现我们的网络请求的服务接口，实现网络请求方法
- ### 3、当代理类对象方法调用时，通过InvocationHandler. invoke方法进行回调
- ### 4、所以我们通过retrofit.create拿到代理类对象时，去调用网络请求方法，都会调用到invoke方法进行处理。
-
- 详细见[[create 源码及过程 2.9.0]]