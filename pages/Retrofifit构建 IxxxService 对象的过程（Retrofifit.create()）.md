title:: Retrofifit构建 IxxxService 对象的过程（Retrofifit.create()）

- ## 使用时
	- ```java
	  ISharedListService sharedListService = retrofit.create(ISharedListService.class);
	  Call<SharedListBean> sharedListCall = sharedListService.getSharedList(2,1);
	  ```
- > 上面两行代码需要连起来才能正确的被阅读，因为在create里面是使用了动态代理的技术方案，而动态代理是运行时生效的，当我们看到create的时候只是创建了代理对象
- ## [[create 源码及过程 2.9.0]]：CallAdapter对OkhttpCall包装返回ExecutorCallBackCall,带执行返回的call