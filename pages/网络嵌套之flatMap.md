- # 网络嵌套错误写法:直接嵌套用rxjava
  collapsed:: true
	- ```java
	      /**
	       * RxJava
	       * RxJs
	       * Rxxxxx
	       * RxBinding  防抖
	       *
	       * TODO 功能防抖 + 网络嵌套（这种是负面教程，嵌套的太厉害了）
	       * 2层嵌套
	       * 6层
	       */
	      @SuppressLint("CheckResult")
	      private void antiShakeActon() {
	          // 注意：（项目分类）查询的id，通过此id再去查询(项目列表数据)
	  
	          // 对那个控件防抖动？
	          Button bt_anti_shake = findViewById(R.id.bt_anti_shake);
	  
	          RxView.clicks(bt_anti_shake)
	                  .throttleFirst(2000, TimeUnit.MILLISECONDS) // 2秒钟之内 响应你一次
	                  .subscribe(new Consumer<Object>() {
	                      @Override
	                      public void accept(Object o) throws Exception {
	                          api.getProject() // 查询主数据
	                          .compose(DownloadActivity.rxud())
	                          .subscribe(new Consumer<ProjectBean>() {
	                              @Override
	                              public void accept(ProjectBean projectBean) throws Exception {
	                                  for (ProjectBean.DataBean dataBean : projectBean.getData()) { // 10
	                                      // 查询item数据
	                                      api.getProjectItem(1, dataBean.getId())
	                                      .compose(DownloadActivity.rxud())
	                                      .subscribe(new Consumer<ProjectItem>() {
	                                          @Override
	                                          public void accept(ProjectItem projectItem) throws Exception {
	                                              Log.d(TAG, "accept: " + projectItem); // 可以UI操作
	                                          }
	                                      });
	                                  }
	                              }
	                          });
	                      }
	                  });
	      }
	  ```
- # [[flatMap]]
	- ```java
	      /**
	       * TODO 功能防抖 + 网络嵌套 (解决嵌套的问题) flatMap
	       */
	      @SuppressLint("CheckResult")
	      private void antiShakeActonUpdate() {
	          // 注意：项目分类查询的id，通过此id再去查询(项目列表数据)
	  
	          // 对那个控件防抖动？
	          Button bt_anti_shake = findViewById(R.id.bt_anti_shake);
	  
	          RxView.clicks(bt_anti_shake)
	                  .throttleFirst(2000, TimeUnit.MILLISECONDS) // 2秒钟之内 响应你一次
	  
	                  // 我只给下面 切换 异步
	                  .observeOn(Schedulers.io())
	                  .flatMap(new Function<Object, ObservableSource<ProjectBean>>() {
	                      @Override
	                      public ObservableSource<ProjectBean> apply(Object o) throws Exception {
	                          return api.getProject(); // 主数据
	                      }
	                  })
	  
	                  // 第一步不能map 因为  api Observbale<Bean>  
	                  /*.map(new Function<Object, ObservableSource<ProjectBean>>() {
	                      @Override
	                      public ObservableSource<ProjectBean> apply(Object o) throws Exception {
	                          return api.getProject(); // 主数据;
	                      }
	                  })*/
	  
	                  .flatMap(new Function<ProjectBean, ObservableSource<ProjectBean.DataBean>>() {
	                      @Override // 我自己搞一个发射器 发多次 10
	                      public ObservableSource<ProjectBean.DataBean> apply(ProjectBean projectBean) throws Exception {
	                          return Observable.fromIterable(projectBean.getData()); 
	                      }
	                  })
	                  .flatMap(new Function<ProjectBean.DataBean, ObservableSource<ProjectItem>>() {
	                      @Override
	                      public ObservableSource<ProjectItem> apply(ProjectBean.DataBean dataBean) throws Exception {
	                          return api.getProjectItem(1, dataBean.getId());
	                      }
	                  })
	  
	                  .observeOn(AndroidSchedulers.mainThread()) // 给下面切换 主线程
	                  .subscribe(new Consumer<ProjectItem>() {
	                      @Override
	                      public void accept(ProjectItem projectItem) throws Exception {
	                          // 如果我要更新UI  会报错2  不会报错1
	                          Log.d(TAG, "accept: " + projectItem);
	                      }
	                  });
	      }
	  ```