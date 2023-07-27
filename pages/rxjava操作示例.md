- ```java
    // TODO *****************************************************  RxJava　思想思维编程
  
      /**
       * ObservableTransformer控制上下游
       * 封装我们的操作:比如线程调度
       * u:upstream   上游
       * d:DownStream 下游
       */
      public final static <UD> ObservableTransformer<UD, UD> rxud() {
          return new ObservableTransformer<UD, UD>() {
              @Override
              public ObservableSource<UD> apply(Observable<UD> upstream) {
                  return  upstream.subscribeOn(Schedulers.io())     // 给上面代码分配异步线程
                  .observeOn(AndroidSchedulers.mainThread()) // 给下面代码分配主线程;
                  .map(new Function<UD, UD>() {
                      @Override
                      public UD apply(UD ud) throws Exception {
                          Log.d(TAG, "apply: 我监听到你了，居然再执行");
                          return ud;
                      }
                  });
                  // .....        ;
              }
          };
      }
  
      public void rxJavaDownloadImageAction(View view) {
          // 起点
          Observable.just(PATH)  // 内部会分发  PATH Stirng  // TODO 第二步
  
           // TODO 第三步
          .map(new Function<String, Bitmap>() {
              @Override
              public Bitmap apply(String s) throws Exception {
                  URL url = new URL(PATH);
                  HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
                  httpURLConnection.setConnectTimeout(5000);
                  int responseCode = httpURLConnection.getResponseCode(); // 才开始 request
                  if (responseCode == HttpURLConnection.HTTP_OK) {
                      InputStream inputStream = httpURLConnection.getInputStream();
                      Bitmap bitmap = BitmapFactory.decodeStream(inputStream);
                      return bitmap;
                  }
                  return null;
              }
          })
          /*.map(new Function<Bitmap, Bitmap>() {
              @Override
              public Bitmap apply(Bitmap bitmap) throws Exception {
                  Paint paint = new Paint();
                  paint.setTextSize(88);
                  paint.setColor(Color.RED);
                  return drawTextToBitmap(bitmap, "同学们大家好",paint, 88 , 88);
              }
          })*/
          
          // 日志记录
          .map(new Function<Bitmap, Bitmap>() {
              @Override
              public Bitmap apply(Bitmap bitmap) throws Exception {
                  Log.d(TAG, "apply: 是这个时候下载了图片啊:" + System.currentTimeMillis());
                  return bitmap;
              }
          })
          .compose(rxud())
  
          // 订阅 起点 和 终点 订阅起来
          .subscribe(
  
                  // 终点
                  new Observer<Bitmap>() {
  
                      // 订阅开始
                      @Override
                      public void onSubscribe(Disposable d) {
                          // 预备 开始 要分发
                          // TODO 第一步
                          progressDialog = new ProgressDialog(DownloadActivity.this);
                          progressDialog.setTitle("download run");
                          progressDialog.show();
                      }
  
                      // TODO 第四步
                      // 拿到事件
                      @Override
                      public void onNext(Bitmap bitmap) {
                          image.setImageBitmap(bitmap);
                      }
  
                      // 错误事件
                      @Override
                      public void onError(Throwable e) {
  
                      }
  
                      // TODO 第五步
                      // 完成事件
                      @Override
                      public void onComplete() {
                          if (progressDialog != null)
                              progressDialog.dismiss();
                      }
          });
  
      }
  
  
      // 图片上绘制文字 加水印
      private final Bitmap drawTextToBitmap(Bitmap bitmap, String text, Paint paint, int paddingLeft, int paddingTop) {
          Bitmap.Config bitmapConfig = bitmap.getConfig();
  
          paint.setDither(true); // 获取跟清晰的图像采样
          paint.setFilterBitmap(true);// 过滤一些
          if (bitmapConfig == null) {
              bitmapConfig = Bitmap.Config.ARGB_8888;
          }
          bitmap = bitmap.copy(bitmapConfig, true);
          Canvas canvas = new Canvas(bitmap);
  
          canvas.drawText(text, paddingLeft, paddingTop, paint);
          return bitmap;
      }
  
  ```